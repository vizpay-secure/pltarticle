# Shrimp Say 🦐

## Category

Web

## Author

Dav3n

## Description
```
 _______________
< Hello, world! >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

A new version of cowsay has been released ! Shrimp Say 🦐 ! 

In this challenge we are given
- The Shrimp-Say website
- A Netcat tunnel linked to a bot that go on the url we want (No internet in this case)

# Methodology
## 1 - Review the code

![Codetree](codetree.png)

We begin by searching where the flag is stored, using for exemple : 
```sh
grep -r flag
```
We can see in the bot.js file that the flag is set in the LocalStorage 
```js
...
  logMainInfo(`Setting the flag in the localStorage for ${CHALLENGE_HOST}...`);
  await page.goto(CHALLENGE_HOST, { timeout: 1000, waitUntil: "domcontentloaded" });

  await page.evaluate((flag) => {
    localStorage.setItem("flag", flag);
  }, FLAG);
...
```
From this, we can determine that: : 
- The flag is stored in the Localstorage of the bot 
- We can submit an url that the bot will visit

## 2 - Enumerate potential vulnerabilities

Next we examine the Dockerfiles in order to see the versions of installed packages
Shrimp-say Dockerfile : 
```docker
FROM php:8.3.0-apache
COPY ./index.php ./shrimp.gif /var/www/html/
EXPOSE 80
```

Shrimp-say-bot Dockerfile : 

```docker
FROM alpine:3.21
WORKDIR /usr/app
COPY ./src/package.json .
RUN apk add --update --no-cache    \
	nodejs~=22                     \
	npm~=10                        \
	socat~=1.8                     \
	chromium-chromedriver~=133  && \
	npm install
COPY ./src .
EXPOSE 4000
CMD ["socat", "tcp-listen:4000,reuseaddr,fork", "exec:'node /usr/app/bot.js'"]
```
After reviewing all that, we see that there are no evident CVE's for this challenge and we start exploring the code.

We understand that the webapp take two parameters **$msg** and **$bg** that are both reflected into the web page.

**$bg** is reflected into the css defining the background color and is passed into htmlentities (sanitizing it).
```css
...
body {
      background-color: <?= htmlentities($bg) ?>;
    }
...
```
And **$msg** is reflected by a strange way (passed in php function base64_encode then gets base64 decoded using atob in js)
```html
<script type="text/base64" id="data"><?= base64_encode($msg) ?></script>
  <script>
    document.querySelector(".speech-bubble").innerHTML = atob(document.getElementById('data').innerText)
  </script>
```
We also take note that the character "<" is forbidden in the **$msg** parameter 

```php
if (strpos($msg, "<") !== false) {
  redirect("NO XSS", "red");
}
```
In conclusion we have two reflected parameters : 
- **$bg**, reflected in a style tag that is sanitized using htmlentities
- **$msg**, directly reflected that is not sanitized but the less-than character is blocked


## 3 - Elaborate strategies
After reviewing everything we have we can think about many attacks, **$bg** is reflected in the style tag so we can go for a css injection but it won't help here because css injection cannot access LocalStorage objects and can only access some direct html data on the page and **$msg** would be a good candidate for xss but "<" is blocked.

So we need to bypass the less-than character restriction in order to achieve the xss here !

My first strategies was to use html entities in order to bypass the restriction ( ```&lt;```or ```&#60;``` ) so the url looked like that 

```https://shrimp-say.fcsc.fr/?msg=&#60;img src=x onerror=alert(1)&#62;&bg=blue```

with the html entities url-encoded for the webapp to not interpret it as a parameter 

These first strategies didn't worked well as the webapp was rendering the script that in a sanitized form.

But next i remembered that i had 2 reflected parameters the **$bg** parameter could control CSS and i watched carefuly all the css properties i was able could use.
We also remember that "<" is blocked ! Why would the admin block this less-than caracter if it wasn't harmful. 
So from here i concluded that the idea was to bypass this restriction.
# Solution

In this challenge, the objective is quite clear, we need to achieve an xss in order to get the flag via the bot with a console.log. The ability to add css via the **$bg** parameter in addition to unsanitized data via the **$msg** parameter

```html
<script type="text/base64" id="data"><?= base64_encode($msg) ?></script>
  <script>
    document.querySelector(".speech-bubble").innerHTML = atob(document.getElementById('data').innerText)
  </script>
```
By looking more carefuly at the code that decode the base64 and render the content we see something really interesting. The uses of innerText here is very interesting because innerText attribute not only get the raw text in the data element, but it get the text by it's appearance.

After looking at all the interesting css properties there's one caught my eye : ```text-transform```, with this property we could use something like ```text-transform: uppercase``` and it would change the base64 data (and it's decoded text) by putting it in uppercase.

So now we have the concept and we need to apply it to bypass the filtered less-than character, we can manipulate the base64 to get a different rendered content that the one we send.

### An exemple : 
If we send as the payload in the **$msg** parameter the text "rb" and we transform the base64 to lowercase (```https://shrimp-say.fcsc.fr/?msg=rb&bg=lightblue);%7D%20%23data%7Bdisplay:block;text-transform:lowercase```) we get the text "rh" in the speech bubble. 

This shows that we can manipulate the inserted data after it passes the strpos check !

### Now let's get the flag !

The first challenge we encounter is that when applying ```text-transform: uppercase``` or ```text-transform: lowercase``` on the **$msg** data it will affect all the base64 data and not only a particular character to change it to "<" so we need to find a way to only change the first character of the base64 to transform the start of the text from a specific character this famous less-than character.

 The solution for that is to use the pseudo-element : ```first-character``` 
 it will allow us to only transform the first character of the base64.
 So to finish and get the flag we need to find a character that, when base64 encoded and transformed to uppercase or lowercase becomes "<".
 We know that the base64 equivalent of ```<``` is ```PA==``` so we need to use the character that as for base64 equivalent ```pA==``` and it corresponds to  the character : ```%4a```.

 We now try to get the flag by using the console.log(localStorage.getItem('flag')) as we know that the flag is stored in the LocalStorage and the bot shows the console.log outputs

 #### Final Payload : 

 ```http://shrimp-say/?msg=%a4img%20src=x%20onerror=console.log(localStorage.getItem(%22flag%22))%3E&bg=lightblue);}%20%23data{display:block;}%20%23data::first-letter{text-transform:uppercase```

 With %4a that get his base64 capitalized and get transformed to < the image is injected and the not found image causes the javascript to execute and the flag to be shown ! (We use shrimp-say host as specified in the docker-compose file)

# Flag

FCSC{f6e865cb389605d91470af3b8555e4535463a1a56157c16c858fa8e9c5ff4513}

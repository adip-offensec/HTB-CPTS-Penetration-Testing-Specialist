
## 1.Introduction to Password Cracking

* Passwords are commonly `hashed` when stored, in order to provide some protection in the event they fall into the hands of an attacker. `Hashing` is a mathematical function which transforms an arbitrary number of input bytes into a (typically) fixed-size output; common examples of hash functions are `MD5`, and `SHA-256`.

```shell-session
echo -n Soccer06! | md5sum
40291c1d19ee11a7df8495c4cccefdfa  -

 echo -n Soccer06! | sha256sum
a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -
```

### Rainbow tables

Rainbow tables are large pre-compiled maps of input and output values for a given hash function. These can be used to very quickly identify the password if its corresponding hash has already been mapped.

| Password   | MD5 Hash                         |
| ---------- | -------------------------------- |
| 123456     | e10adc3949ba59abbe56e057f20f883e |
| 12345      | 827ccb0eea8a706c4c34a16891f84e7b |
| 123456789  | 25f9e794323b453885f5181f1b624d0b |
| password   | 5f4dcc3b5aa765d61d8327deb882cf99 |
| iloveyou   | f25a2fc72690b780b2a14e140ef6a9e0 |
| princess   | 8afa847f50a716e64932d995c8e7435a |
| 1234567    | fcea920f7412b5da7be0cf42b8c93759 |
| rockyou    | f806fc5a2a0d5ba2471600758452799c |
| 12345678   | 25d55ad283aa400af464c76d713c07ad |
| abc123     | e99a18c428cb38d5f260853678922e03 |
| ...SNIP... | ...SNIP...                       |

* Because rainbow tables are such a powerful attack, `salting` is used. A `salt`, in cryptographic terms, is a random sequence of bytes added to a password before it is hashed.

```shell-session
echo -n Th1sIsTh3S@lt_Soccer06! | md5sum

90a10ba83c04e7996bc53373170b5474  -
```

### Brute-force attack

| Brute-force attempt | MD5 Hash                         |
| ------------------- | -------------------------------- |
| ...SNIP...          | ...SNIP...                       |
| Sxejd               | 2cdc813ef26e6d20c854adb107279338 |
| Sxeje               | 7703349a1f943f9da6d1dfcda51f3b63 |
| Sxejf               | db914f10854b97946046eabab2287178 |
| Sxejg               | c0ceb70c0e0f2c3da94e75ae946f29dc |
| Sxejh               | 4dca0d2b706e9344985d48f95e646ce8 |
| Sxeji               | 66b5fa128df895d50b2d70353a7968a7 |
| Sxejj               | dd7097ba514c136caac321e321b1b5ca |
| Sxejk               | c0eb1193e62a7a57dec2fafd4177f7d9 |
| Sxejl               | 5ad8e1282437da255b866d22339d1b53 |
| Sxejm               | c4b95c1fe6d2a4f22620efd54c066664 |
| ...SNIP...          | ...SNIP...                       |

### Dictionary attack

* seclists can be also used

```shell-session
locate rockyou.txt
gunzip rockyou.tar.gz
head --lines=20 /usr/share/wordlists/rockyou.txt 
```

## 2.John The Ripper

### Cracking modes

`Single crack mode` is a rule-based cracking technique that is most useful when targeting Linux credentials. It generates password candidates based on the victim's username, home directory name, and [GECOS](https://en.wikipedia.org/wiki/Gecos_field) values (full name, room number, phone number, etc.).&#x20;

***

* Imagine we as attackers came across the file `passwd` with the following contents:

```
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```

```shell-session
john --single passwd
```

**Wordlist mode**

`Wordlist mode` is used to crack passwords with a dictionary attack, meaning it attempts all passwords in a supplied wordlist against the password hash.&#x20;

```shell-session
john --wordlist=<wordlist_file> <hash_file>
```

**Incremental mode**

`Incremental mode` is a powerful, brute-force-style password cracking mode that generates candidate passwords based on a statistical model ([Markov chains](https://en.wikipedia.org/wiki/Markov_chain)).

```shell-session
 john --incremental <hash_file>
```

```shell-session
grep '# Incremental modes' -A 100 /etc/john/john.confd
```

### Identifying hash formats

* <https://openwall.info/wiki/john/sample-hashes>
* <https://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats>
* <https://github.com/psypanda/hashID>

```shell-session
hashid -j 193069ceb0461e1d40d216e32c79c704
```

The `--format` argument can be supplied to instruct JtR which format target hashes have.

| **Hash format**      | **Example command**                               | **Description**                                                      |
| -------------------- | ------------------------------------------------- | -------------------------------------------------------------------- |
| afs                  | `john --format=afs [...] <hash_file>`             | AFS (Andrew File System) password hashes                             |
| bfegg                | `john --format=bfegg [...] <hash_file>`           | bfegg hashes used in Eggdrop IRC bots                                |
| bf                   | `john --format=bf [...] <hash_file>`              | Blowfish-based crypt(3) hashes                                       |
| bsdi                 | `john --format=bsdi [...] <hash_file>`            | BSDi crypt(3) hashes                                                 |
| crypt(3)             | `john --format=crypt [...] <hash_file>`           | Traditional Unix crypt(3) hashes                                     |
| des                  | `john --format=des [...] <hash_file>`             | Traditional DES-based crypt(3) hashes                                |
| dmd5                 | `john --format=dmd5 [...] <hash_file>`            | DMD5 (Dragonfly BSD MD5) password hashes                             |
| dominosec            | `john --format=dominosec [...] <hash_file>`       | IBM Lotus Domino 6/7 password hashes                                 |
| EPiServer SID hashes | `john --format=episerver [...] <hash_file>`       | EPiServer SID (Security Identifier) password hashes                  |
| hdaa                 | `john --format=hdaa [...] <hash_file>`            | hdaa password hashes used in Openwall GNU/Linux                      |
| hmac-md5             | `john --format=hmac-md5 [...] <hash_file>`        | hmac-md5 password hashes                                             |
| hmailserver          | `john --format=hmailserver [...] <hash_file>`     | hmailserver password hashes                                          |
| ipb2                 | `john --format=ipb2 [...] <hash_file>`            | Invision Power Board 2 password hashes                               |
| krb4                 | `john --format=krb4 [...] <hash_file>`            | Kerberos 4 password hashes                                           |
| krb5                 | `john --format=krb5 [...] <hash_file>`            | Kerberos 5 password hashes                                           |
| LM                   | `john --format=LM [...] <hash_file>`              | LM (Lan Manager) password hashes                                     |
| lotus5               | `john --format=lotus5 [...] <hash_file>`          | Lotus Notes/Domino 5 password hashes                                 |
| mscash               | `john --format=mscash [...] <hash_file>`          | MS Cache password hashes                                             |
| mscash2              | `john --format=mscash2 [...] <hash_file>`         | MS Cache v2 password hashes                                          |
| mschapv2             | `john --format=mschapv2 [...] <hash_file>`        | MS CHAP v2 password hashes                                           |
| mskrb5               | `john --format=mskrb5 [...] <hash_file>`          | MS Kerberos 5 password hashes                                        |
| mssql05              | `john --format=mssql05 [...] <hash_file>`         | MS SQL 2005 password hashes                                          |
| mssql                | `john --format=mssql [...] <hash_file>`           | MS SQL password hashes                                               |
| mysql-fast           | `john --format=mysql-fast [...] <hash_file>`      | MySQL fast password hashes                                           |
| mysql                | `john --format=mysql [...] <hash_file>`           | MySQL password hashes                                                |
| mysql-sha1           | `john --format=mysql-sha1 [...] <hash_file>`      | MySQL SHA1 password hashes                                           |
| NETLM                | `john --format=netlm [...] <hash_file>`           | NETLM (NT LAN Manager) password hashes                               |
| NETLMv2              | `john --format=netlmv2 [...] <hash_file>`         | NETLMv2 (NT LAN Manager version 2) password hashes                   |
| NETNTLM              | `john --format=netntlm [...] <hash_file>`         | NETNTLM (NT LAN Manager) password hashes                             |
| NETNTLMv2            | `john --format=netntlmv2 [...] <hash_file>`       | NETNTLMv2 (NT LAN Manager version 2) password hashes                 |
| NEThalfLM            | `john --format=nethalflm [...] <hash_file>`       | NEThalfLM (NT LAN Manager) password hashes                           |
| md5ns                | `john --format=md5ns [...] <hash_file>`           | md5ns (MD5 namespace) password hashes                                |
| nsldap               | `john --format=nsldap [...] <hash_file>`          | nsldap (OpenLDAP SHA) password hashes                                |
| ssha                 | `john --format=ssha [...] <hash_file>`            | ssha (Salted SHA) password hashes                                    |
| NT                   | `john --format=nt [...] <hash_file>`              | NT (Windows NT) password hashes                                      |
| openssha             | `john --format=openssha [...] <hash_file>`        | OPENSSH private key password hashes                                  |
| oracle11             | `john --format=oracle11 [...] <hash_file>`        | Oracle 11 password hashes                                            |
| oracle               | `john --format=oracle [...] <hash_file>`          | Oracle password hashes                                               |
| pdf                  | `john --format=pdf [...] <hash_file>`             | PDF (Portable Document Format) password hashes                       |
| phpass-md5           | `john --format=phpass-md5 [...] <hash_file>`      | PHPass-MD5 (Portable PHP password hashing framework) password hashes |
| phps                 | `john --format=phps [...] <hash_file>`            | PHPS password hashes                                                 |
| pix-md5              | `john --format=pix-md5 [...] <hash_file>`         | Cisco PIX MD5 password hashes                                        |
| po                   | `john --format=po [...] <hash_file>`              | Po (Sybase SQL Anywhere) password hashes                             |
| rar                  | `john --format=rar [...] <hash_file>`             | RAR (WinRAR) password hashes                                         |
| raw-md4              | `john --format=raw-md4 [...] <hash_file>`         | Raw MD4 password hashes                                              |
| raw-md5              | `john --format=raw-md5 [...] <hash_file>`         | Raw MD5 password hashes                                              |
| raw-md5-unicode      | `john --format=raw-md5-unicode [...] <hash_file>` | Raw MD5 Unicode password hashes                                      |
| raw-sha1             | `john --format=raw-sha1 [...] <hash_file>`        | Raw SHA1 password hashes                                             |
| raw-sha224           | `john --format=raw-sha224 [...] <hash_file>`      | Raw SHA224 password hashes                                           |
| raw-sha256           | `john --format=raw-sha256 [...] <hash_file>`      | Raw SHA256 password hashes                                           |
| raw-sha384           | `john --format=raw-sha384 [...] <hash_file>`      | Raw SHA384 password hashes                                           |
| raw-sha512           | `john --format=raw-sha512 [...] <hash_file>`      | Raw SHA512 password hashes                                           |
| salted-sha           | `john --format=salted-sha [...] <hash_file>`      | Salted SHA password hashes                                           |
| sapb                 | `john --format=sapb [...] <hash_file>`            | SAP CODVN B (BCODE) password hashes                                  |
| sapg                 | `john --format=sapg [...] <hash_file>`            | SAP CODVN G (PASSCODE) password hashes                               |
| sha1-gen             | `john --format=sha1-gen [...] <hash_file>`        | Generic SHA1 password hashes                                         |
| skey                 | `john --format=skey [...] <hash_file>`            | S/Key (One-time password) hashes                                     |
| ssh                  | `john --format=ssh [...] <hash_file>`             | SSH (Secure Shell) password hashes                                   |
| sybasease            | `john --format=sybasease [...] <hash_file>`       | Sybase ASE password hashes                                           |
| xsha                 | `john --format=xsha [...] <hash_file>`            | xsha (Extended SHA) password hashes                                  |
| zip                  | `john --format=zip [...] <hash_file>`             | ZIP (WinZip) password hashes                                         |

## Cracking files

```shell-session
<tool> <file_to_crack> > file.hash
```

```shell-session
locate *2john*
```

Some of the tools included with JtR are:

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
| ...SNIP...              | ...SNIP...                                    |

## 3.Hashcat

```shell-session
 hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

> In the command above:
>
> * `-a` is used to specify the `attack mode`
> * `-m` is used to specify the `hash type`
> * `<hashes>` is a either a hash string, or a file containing one or more password hashes of the same type
> * `[wordlist, rule, mask, ...]` is a placeholder for additional arguments that depend on the attack mode

### Hash types

```shell-session
hashcat --help
```

Alternatively, [hashID](https://github.com/psypanda/hashID) can be used to quickly identify the hashcat hash type by specifying the `-m` argument.

```shell-session
hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
```

## Attack modes

### **Dictionary attack**

* As an example, imagine we extracted the following password hash from an SQL database: `e3e3ec5831ad5e7288241960e5d4fdb8`. First, we could identify this as an MD5 hash, which has a hash ID of `0`. To attempt to crack this hash using the `rockyou.txt` wordlist

```shell-session
hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
```

```shell-session
ls -l /usr/share/hashcat/rules
```

* we could try is `best64.rule`

```shell-session
hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

### **Mask attack**

A mask is defined by combining a sequence of symbols, each representing a built-in or custom character set. Hashcat includes several built-in character sets:

| Symbol | Charset                                 |
| ------ | --------------------------------------- |
| ?l     | abcdefghijklmnopqrstuvwxyz              |
| ?u     | ABCDEFGHIJKLMNOPQRSTUVWXYZ              |
| ?d     | 0123456789                              |
| ?h     | 0123456789abcdef                        |
| ?H     | 0123456789ABCDEF                        |
| ?s     | «space»!"#$%&'()\*+,-./:;<=>?@\[]^\_\`{ |
| ?a     | ?l?u?d?s                                |
| ?b     | 0x00 - 0xff                             |

Custom charsets can be defined with the `-1`, `-2`, `-3`, and `-4` arguments, then referred to with `?1`, `?2`, `?3`, and `?4`.

Let's say that we specifically want to try passwords which start with an uppercase letter, continue with four lowercase letters, a digit, and then a symbol. The resulting hashcat mask would be `?u?l?l?l?l?d?s`.

```shell-session
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

## 4.Writing Custom Wordlists and Rules

```shell-session
cat password.list
cat custom.rule
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
cat mut_password.list
```

Hashcat and JtR both come with pre-built rule lists that can be used for password generation and cracking. One of the most effective and widely used rulesets is `best64.rule`, which applies common transformations that frequently result in successful password guesses.

## Generating wordlists using CeWL

```shell-session
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
wc -l inlane.wordlist
```

## 5.Cracking Protected Files

### Hunting for Encrypted Files

* <https://fileinfo.com/filetypes/encoded>

```shell-session
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

### Hunting for SSH keys

```shell-session
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

```shell-session
cat /home/jsmith/.ssh/SSH.private
```

* One way to tell whether an SSH key is encrypted or not, is to try reading the key with `ssh-keygen`.

```shell-session
ssh-keygen -yf ~/.ssh/id_ed25519 

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIpNefJd834VkD5iq+22Zh59Gzmmtzo6rAffCx2UtaS6



ssh-keygen -yf ~/.ssh/id_rsa

Enter passphrase for "/home/jsmith/.ssh/id_rsa":
```

### Cracking encrypted SSH keys

```shell-session
locate *2john*
```

* we could use the Python script `ssh2john.py` to acquire the corresponding hash for an encrypted SSH key, and then use JtR to try and crack it.

```shell-session
ssh2john.py SSH.private > ssh.hash
john --wordlist=rockyou.txt ssh.hash
```

We can then view the resulting hash:

```shell-session
john ssh.hash --show
```

### Cracking password-protected documents

```shell-session
office2john.py Protected.docx > protected-docx.hash
john --wordlist=rockyou.txt protected-docx.hash
john protected-docx.hash --show
```

The process for cracking PDF files is quite similar, as we simply swap out `office2john.py` for `pdf2john.py`.

```shell-session
pdf2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
john pdf.hash --show
```

## 6.Cracking Protected Archives

Besides standalone files, we will often run across `archives` and `compressed files`—such as ZIP files—which are protected with a password.

There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

* <https://fileinfo.com/filetypes/compressed>

```shell-session
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

* TAR files are commonly encrypted using `openssl` or `gpg`.

### Cracking ZIP files

```shell-session
zip2john ZIP.zip > zip.hash
cat zip.hash 
```

* Once we have extracted the hash, we can use JtR to crack it with the desired password list.

```shell-session
john --wordlist=rockyou.txt zip.hash
john zip.hash --show
```

### Cracking OpenSSL encrypted GZIP files

```shell-session
file GZIP.gzip 
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;don
```

* Once the `for` loop has finished, we can check the current directory for a newly extracted file.

```shell-session
ls

customers.csv  GZIP.gzip  rockyou.txt
```

### Cracking BitLocker-encrypted drives

[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows operating system. Available since Windows Vista, it uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths. If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john` to [four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker): the first two correspond to the BitLocker password, while the latter two represent the recovery key. Because the recovery key is very long and randomly generated, it is generally not practical to guess—unless partial knowledge is available. Therefore, we will focus on cracking the password using the first hash (`$bitlocker$0$...`).

```shell-session
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash
```

```shell-session
hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt
```

After successfully cracking the password, we can access the encrypted drive.

**Mounting BitLocker-encrypted drives in Windows**

<figure><img src="https://27438919-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FXIPxco87mgI3Ma3yEBLL%2Fuploads%2F2hBgjKBTnNi5mh3RdWBn%2Fimage.png?alt=media&amp;token=0d460199-647b-4928-bedd-43e307e9dc37" alt=""><figcaption></figcaption></figure>

## Mounting BitLocker-encrypted drives in Linux (or macOS)

{% embed url="<https://github.com/Aorimn/dislocker>" %}

```shell-session
sudo apt-get install dislocker
```

* Next, we create two folders which we will use to mount the VHD.

```shell-session
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
```

We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:

```shell-session
sudo losetup -f -P Backup.vhd
sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

* If everything was done correctly, we can now browse the files:

```shell-session
cd /media/bitlockermount/
ls -la
```

* Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

```shell-session
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

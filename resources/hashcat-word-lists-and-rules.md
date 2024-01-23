# Hashcat Word lists and Rules

## Recommended General Large Word lists

<table><thead><tr><th width="398">Word List</th><th>Link</th></tr></thead><tbody><tr><td>AllInOne</td><td><a href="https://weakpass.com/all-in-one">https://weakpass.com/all-in-one</a></td></tr><tr><td>Rockyou2021</td><td><a href="https://weakpass.com/wordlist/1943">https://weakpass.com/wordlist/1943</a></td></tr><tr><td>Weakpass_3a</td><td><a href="https://weakpass.com/wordlist/1948">https://weakpass.com/wordlist/1948</a></td></tr><tr><td>Top2Billion-probable-v2</td><td><a href="https://weakpass.com/wordlist/1858">https://weakpass.com/wordlist/1858</a></td></tr></tbody></table>

## Recommended General Medium Word lists

<table><thead><tr><th width="399">Word List</th><th>Link</th></tr></thead><tbody><tr><td>hk_hlm_founds</td><td><a href="https://weakpass.com/wordlist/1256">https://weakpass.com/wordlist/1256</a></td></tr><tr><td>RP4</td><td><a href="https://weakpass.com/wordlist/914">https://weakpass.com/wordlist/914</a></td></tr><tr><td>Ignis</td><td><a href="https://weakpass.com/wordlist/1935">https://weakpass.com/wordlist/1935</a></td></tr><tr><td>Top29Million-probable-v2</td><td><a href="https://weakpass.com/wordlist/1857">https://weakpass.com/wordlist/1857</a></td></tr><tr><td>SkullSecurityComp</td><td><a href="https://weakpass.com/wordlist/671">https://weakpass.com/wordlist/671</a></td></tr></tbody></table>

## Specific Word lists

<table><thead><tr><th width="170">Word List</th><th width="228">Use case</th><th>Link</th></tr></thead><tbody><tr><td>Kerberoast_pws</td><td>SPN cracking</td><td><a href="https://gist.github.com/The-Viper-One/a1ee60d8b3607807cc387d794e809f0b">https://gist.github.com/The-Viper-One/a1ee60d8b3607807cc387d794e809f0b</a></td></tr><tr><td>weakpass_3w</td><td>8-24 characters</td><td><a href="https://weakpass.com/wordlist/1950">https://weakpass.com/wordlist/1950</a></td></tr><tr><td>weakpass_3p</td><td>Contains only printable characters</td><td><a href="https://weakpass.com/wordlist/1949">https://weakpass.com/wordlist/1949</a></td></tr></tbody></table>

## Word list from cracked hashes

Locate pot-file

```
find / -name hashcat.potfile 2> /dev/null
```

Place the cracked hash passwords into its own word list.

```
cat [PotFile] | sed 's/[^:]*://' > CrackedHashesWordlist.txt
```

## Word list from website scraping

```
cewl [URL] -d 3 -m 5 --with-numbers | tee Wordlists/CewlWordList.txt
```

## Recommended Rules

#### NSA Rules

Github: [https://github.com/NSAKEY/nsa-rules](https://github.com/NSAKEY/nsa-rules)

```
git clone https://github.com/NSAKEY/nsa-rules.git
```

#### OneRuleToRuleThemAllStill

An updated and improved variation of the popular OneRuleToRuleThemAll rule set. This updated rule set should provide the same effective crackrate as OneRule with a reduction in total cracking time.

Blog Post: [https://in.security/2023/01/10/oneruletorulethemstill-new-and-improved/](https://in.security/2023/01/10/oneruletorulethemstill-new-and-improved/)

Github: [https://github.com/stealthsploit/OneRuleToRuleThemStill](https://github.com/stealthsploit/OneRuleToRuleThemStill)

```
git clone https://github.com/stealthsploit/OneRuleToRuleThemStill.git
```

#### Unic0rn28 Hashcat Rules

Github: [https://github.com/Unic0rn28/hashcat-rules](https://github.com/Unic0rn28/hashcat-rules)

<pre><code><strong>git clone https://github.com/Unic0rn28/hashcat-rules.git
</strong></code></pre>

## Brute Force Mask

```bash
hashcat -m 13100 -O -a3 ?a?a?a?a?a?a?a?a --increment # Bruteforce all upto 8 characters
```

## Reviewing cracked passwords

Hashcat can display credentials in \[Username]:\[Password] format. Adjust the command below to match the correct method for the hashfile and the --outfile-format value to whichever looks best. For NTLM and Secretsdump the command below should work fine.

```
hashcat -m 1000 SecretsDump.txt --show --username --outfile-format 2 | sort
```

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>


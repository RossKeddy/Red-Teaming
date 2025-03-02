## Google DORKS
```
# Google Hacking made easy
https://pentest-tools.com/information-gathering/google-hacking#

# When you use the Google Dork:  site:*.example.com, NEVER forget to check
site:*.*.example.com
site:*.*.*.example.com 

# Search for documents on popular clouds
site:drive.google.com <searchterm>
site:dl.dropbox.com <searchterm>
site:s3.amazonaws.com <searchterm>
site:onedrive.live.com <searchterm>
site:cryptome.org <searchterm>

# Admins credentials
intext:company_keyword & ext:txt | ext:sql | ext:cnf | ext:config | ext:log & intext:"admin" | intext:"root" | intext:"administrator" & intext:"password" | intext:"root" | intext:"admin" | intext:"administrator"

# Recon to find sensivite data
site:http://ideone.com  | site:http://codebeautify.org  | site:http://codeshare.io  | site:http://codepen.io  | site:http://repl.it  | site:http://justpaste.it  | site:http://pastebin.com  | site:http://jsfiddle.net  | site:http://trello.com  "$TARGET"
```

## GitHub DORKS
```
https://gist.github.com/jhaddix/77253cea49bf4bd4bfd5d384a37ce7a4

# Github dorks work a lot with filename and extension
# You can build search like this
filename:bashrc
extension:pem
langage:bash

# Possible to search terms and use these keywords
# Some usefull examples
extension:pem private # Private SSH Keys
extension:sql mysql dump # MySQL dumps
extension:sql mysql dump password # MySQL dumps with passwords
extension:json mongolab.com # Keys/Credentials for mongolab

filename:wp-config.php # Wordpress config file
filename:.htpasswd # .htpasswd
filename:.git-credentials # Git stored credentials
filename:.bashrc password # .bashrc files containing passwords
filename:.bash_profile aws # AWS keys in .bash_profiles

filename:filezilla.xml Pass # FTP credentials
filename:recentservers.xml Pass # FTP credentials
filename:config.php dbpasswd # PHP Applications databases credentials
shodan_api_key language:python # Shodan API Keys (try others languages)
filename:logins.json # Firefox saved password collection (key3.db usually in same repo)
filename:settings.py SECRET_KEY # Django secret keys (usually allows for session hijacking, RCE, etc)
```
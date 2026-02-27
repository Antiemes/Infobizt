# Jelszavak biztonsága

## SSH publikus kulcs alapú authentikáció

ssh-keygen
.ssh könyvtár
known_hosts
authorized_keys

## Jelszavak generálása

pwgen

## Belépési jelszavak, PAM

/etc/login.defs
PASS_MAX_DAYS
PASS_MIN_DAYS
PASS_WARN_AGE


libpam-pwquality


/etc/pam.d/common-password
password        [success=1 default=ignore]      pam_unix.so obscure use_authtok try_first_pass sha512 remember=5

/etc/security/pwquality.conf

minlen = 8
minclass = 3
maxrepeat = 2
maxclassrepeat = 4
maxsequence = 3
gecoscheck = 1
dictcheck
dictpath
/usr/share/dict/cracklib-small


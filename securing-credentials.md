# Securing credentials

Usually is not nice to type clear passwords in the console as someone might get the history or if someone gets into the server my be able get the clear text password from some file.

The best practice would encrypt it first using GPG keys, so you might create your GPG key first (or use your already existant).

Let's consider your GPG e-mail is `foo@bar`

Create a file with your password inside:

```
cat > .pass <<PWD
this-is-my-pass
PWD
```
Then encrypt the file
```
$ gpg --encrypt -r foo@bar # this will generate the .pass.gpg file
```
Remove the unencrypted file
```
$ rm -f .pass
```

To use decrypt the file content to a variable. Never decrypt to a plain text file.

```
$ export PASS=$(gpg --decrypt .pass.gpg -r foo@bar)
```

That's it!

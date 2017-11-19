In case it is of interest to someone, here is what I did to get it running:

 1. **Create a new GPG key**
 
    I already had one GPG key but with other email.
    Although it is possible to add another email to the key,
    I wanted to separate things, as I am going to use this key
    too many times for signing every commit, and also will add
    its passphrase to the gpg-agent.
    Those things may make the GPG key more easy to compromise.
    
    I have used `gpg2` as is more suitable for destkop environments
    than `gpg` (but the commands are compatible).
    
        gpg2 --full-gen-key
    
    I created a `4096-bit` `RSA/RSA` key that `don't expire` and with
    the name set to the same value of the git config `user.name`, and
    the email set to the git `user.email`. It is important that this
    values are equal and that you don't add no comment to allow git
    to correctly get the suitable key automatically.

2. **Add the key to Github**
    
    [Here](https://github.com/settings/keys) I added the key that I exported with
    
        gpg2 --export <key-id>
    
    The `<key-id>` is the key id of the newly-created key. It is printed when
    the key ends generating. You can also get it with `gpg2 --list-secret-keys`.

3. **Set git to sign by default**

   As I am using `gpg2`, I have to tell git to use it too, as `gpg` won't be able
   to read GPG private keys generated with `gpg2` (the opposite is also true,
   `gpg2` won't see private keys generated with `gpg`, but both of them see the same
   public keys).
   
       git config --global gpg.program gpg2
   
   Now, to sign every commit by default:
   
       git config --global commit.gpgSign true
   
   If you use `git-flow` and you want your release and hotfix tags to be signed as well
   ([taken from here](https://github.com/petervanderdoes/gitflow-avh/wiki/Reference:-Configuration))
   
       git config --global gitflow.release.branch.sign true
       git config --global gitflow.release.finish.sign true
       git config --global gitflow.hotfix.finish.sign true
       
4. **Add the GPG key passphrase to the gpg-agent**
   
   In Ubuntu, I only have to use once the key (`gpg2 --sign --default-key <key-id>`)
   and in the dialog that appears asking me the passphrase for it it will
   have a checkbox to add it to the keyring (ie. remember it).
   So, check it and you won't have to type it anymore while logged into
   the agent (which is logged in automatically when you login on the X session).

5. [OPTIONAL] **Tell git what key to use for signing**
   
   If you followed my advice of naming the key with your git `name` and `email`,
   you don't have to do anything here, as for every sign operation, git will
   look for a key with the format `user.name <user.email>`, and it will match
   your newly created key unless you change them.
   
   I like having it that way, because if you use another user or email for a
   specific repo, git will try to search for a corresponding key for it and
   fail if it doesn't find any. So you will get notified instead of silently
   keeping signing commits with the same key for this repo too.
   
   If you still want to sign commits with the same key having a different
   user or email, you have two options: setting `signinKey` for that specific
   repo (as we will see later), or adding that `user <email>` identity to the
   GPG key.
   
   GPG keys can have multiple identities (ie. name-email combinations),
   and adding one will modify the key id, meaning that you don't have to change
   the key in every site you have uploaded it too (like GitHub). Just add it
   and update the key in the relevant site (the site of the specific repo usually).
   
   Ok, if you don't followed the advice or want to use a key with different name
   or email as your git name and email, or simply want to add a comment to the
   key identify and you don't want to have multiple identities as I have talked
   about before, you can tell git to always use the same key for every sign
   operation with the following command:
   
       git config --global user.signingKey key-id
   
   Remove the `--global` if you want to set it only for the current repository.
   
4. **Commit as usual!**

   All your new commits will be signed, and the tags you create with `git tag -s` too!
   Also, the `git-flow` created tags by `git flow feature finish` and the like
   will be signed too!.
   
   You can confirm in the GitHub web interface that it is working correctly
   by looking at the **Verified** tag of the commits.
   

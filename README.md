mail2rm
=======

mail2rm allows you to send PDF documents to your reMarkable device by email.
The script is intended for people who run an own mail transport agent (MTA)
instance e.g. postfix. mail2rm is supposed to be used as a postfix filter.
This allows you to run the script during mail delivery. If the upload fails you
will get an undeliverable notice including the error message.


Installation
------------

 1. Download rmapi from <https://github.com/juruen/rmapi/releases>
    and copy the extractable binary to your server e.g.
    to ```/usr/local/bin/rmapi```

 2. Add a separate user to your server (here Debian/Ubuntu):

    ```bash
    adduser --system mail2rm
    ```

 3. Copy the mail2rm script to your server e.g. to
    ```/usr/local/bin/mail2rm``` and make it executable.

 4. Register rmapi as the mail2rm user

    ```bash
    sudo -u mail2rm rmapi
    ```

 5. Integrate mail2rm into postfix.

    Add the filter to master.cf:
    ```
    mail2rm unix - n n - 50 pipe flags=F user=mail2rm argv=/usr/local/bin/mail2rm -a mymail@example.com -a workmail@example.com -r /usr/local/bin/rmapi ${sender}
    ```
    The -a/--allowed sender option allows to limit the addresses which can send
    documents. The option can be used multiple times. Replace the above -a
    statemens with your email addresses. The -r/--rmapi option
    specifies the path to the rmapi executable. 

    Add a check_recipient_access rule in master.cf:
    ```
    smtpd_recipient_restrictions =
        [...]
        check_recipient_access hash:/etc/postfix/recipient_access
    ```

    Add the destination address to recipient_access file:
    ```
    remarkable@example.com FILTER mail2rm:dummy
    ```

    Update and reload configuration:
    ```bash
    postmap /etc/postfix/recipient_access
    systemctl reload postfix
    ```

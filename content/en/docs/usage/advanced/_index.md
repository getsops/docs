---
title: "Advanced usage"
weight: 60
description: Advanced SOPS usage.
---

## Encrypting and decrypting from other programs

When using `sops` in scripts or from other programs, there are often
situations where you do not want to write encrypted or decrypted data to
disk. The best way to avoid this is to pass data to SOPS via stdin, and
to let SOPS write data to stdout. By default, the encrypt and decrypt
operations write data to stdout already. To pass data via stdin, you
need to not provide an input filename. For encryption, you also must
provide the `--filename-override` option with the file's filename. The
filename will be used to determine the input and output types, and to
select the correct creation rule.

The simplest way to decrypt data from stdin is as follows:

``` sh
$ cat encrypted-data | sops decrypt > decrypted-data
```

By default, `sops` determines the input and output format from the
provided filename. Since in this case, no filename is provided, `sops`
will use the binary store which expects JSON input and outputs binary
data on decryption. This is often not what you want.

To avoid this, you can either provide a filename with `--filename-override`,
or explicitly control the input and output formats by passing
`--input-type` and `--output-type` as appropriate:

``` sh
$ cat encrypted-data | sops decrypt --filename-override filename.yaml > decrypted-data
$ cat encrypted-data | sops decrypt --input-type yaml --output-type yaml > decrypted-data
```

In both cases, `sops` will assume that the data you provide is in YAML
format, and will encode the decrypted data in YAML as well. The second
form allows to use different formats for input and output.

To encrypt, it is important to note that SOPS also uses the filename to
look up the correct creation rule from `.sops.yaml`. Therefore, you must
provide the `--filename-override` parameter which allows you to tell
SOPS which filename to use to match creation rules:

``` sh
$ echo 'foo: bar' | sops encrypt --filename-override path/filename.sops.yaml > encrypted-data
```

SOPS will find a matching creation rule for `path/filename.sops.yaml` in
`.sops.yaml` and use that one to encrypt the data from stdin. This
filename will also be used to determine the input and output store. As
always, the input store type can be adjusted by passing `--input-type`,
and the output store type by passing `--output-type`:

``` sh
$ echo foo=bar | sops encrypt --filename-override path/filename.sops.yaml --input-type dotenv > encrypted-data
```

## Auditing

Sometimes, users want to be able to tell what files were accessed by
whom in an environment they control. For this reason, SOPS can generate
audit logs to record activity on encrypted files. When enabled, SOPS
will write a log entry into a pre-configured PostgreSQL database when a
file is decrypted. The log includes a timestamp, the username SOPS is
running as, and the file that was decrypted.

In order to enable auditing, you must first create the database and
credentials using the schema found in `audit/schema.sql`. This schema
defines the tables that store the audit events and a role named `sops`
that only has permission to add entries to the audit event tables. The
default password for the role `sops` is `sops`. You should change this
password.

Once you have created the database, you have to tell SOPS how to connect
to it. Because we don\'t want users of SOPS to be able to control
auditing, the audit configuration file location is not configurable, and
must be at `/etc/sops/audit.yaml`. This file should have strict
permissions such that only the root user can modify it.

For example, to enable auditing to a PostgreSQL database named `sops`
running on localhost, using the user `sops` and the password `sops`,
`/etc/sops/audit.yaml` should have the following contents:

``` yaml
backends:
    postgres:
        - connection_string: "postgres://sops:sops@localhost/sops?sslmode=verify-full"
```

You can find more information on the `connection_string` format in the
[PostgreSQL
docs](https://www.postgresql.org/docs/current/static/libpq-connect.html#libpq-connstring).

Under the `postgres` map entry in the above YAML is a list, so one can
provide more than one backend, and SOPS will log to all of them:

``` yaml
backends:
    postgres:
        - connection_string: "postgres://sops:sops@localhost/sops?sslmode=verify-full"
        - connection_string: "postgres://sops:sops@remotehost/sops?sslmode=verify-full"
```

## Passing Secrets to Other Processes

In addition to writing secrets to standard output and to files on disk,
SOPS has two commands for passing decrypted secrets to a new process:
`exec-env` and `exec-file`. These commands will place all output into
the environment of a child process and into a temporary file,
respectively. For example, if a program looks for credentials in its
environment, `exec-env` can be used to ensure that the decrypted
contents are available only to this process and never written to disk.

``` sh
# print secrets to stdout to confirm values
$ sops decrypt out.json
{
        "database_password": "jf48t9wfw094gf4nhdf023r",
        "AWS_ACCESS_KEY_ID": "AKIAIOSFODNN7EXAMPLE",
        "AWS_SECRET_KEY": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}

# decrypt out.json and run a command
# the command prints the environment variable and runs a script that uses it
$ sops exec-env out.json 'echo secret: $database_password; ./database-import'
secret: jf48t9wfw094gf4nhdf023r

# launch a shell with the secrets available in its environment
$ sops exec-env out.json 'sh'
sh-3.2# echo $database_password
jf48t9wfw094gf4nhdf023r

# the secret is not accessible anywhere else
sh-3.2$ exit
$ echo your password: $database_password
your password:
```

If the command you want to run only operates on files, you can use
`exec-file` instead. By default, SOPS will use a FIFO to pass the
contents of the decrypted file to the new program. Using a FIFO, secrets
are only passed in memory which has two benefits: the plaintext secrets
never touch the disk, and the child process can only read the secrets
once. In contexts where this won\'t work, eg platforms like Windows
where FIFOs unavailable or secret files that need to be available to the
child process longer term, the `--no-fifo` flag can be used to instruct
SOPS to use a traditional temporary file that will get cleaned up once
the process is finished executing. `exec-file` behaves similar to
`find(1)` in that `{}` is used as a placeholder in the command which
will be substituted with the temporary file path (whether a FIFO or an
actual file).

``` sh
# operating on the same file as before, but as a file this time
$ sops exec-file out.json 'echo your temporary file: {}; cat {}'
your temporary file: /tmp/.sops894650499/tmp-file
{
        "database_password": "jf48t9wfw094gf4nhdf023r",
        "AWS_ACCESS_KEY_ID": "AKIAIOSFODNN7EXAMPLE",
        "AWS_SECRET_KEY": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}

# launch a shell with a variable TMPFILE pointing to the temporary file
$ sops exec-file --no-fifo out.json 'TMPFILE={} sh'
sh-3.2$ echo $TMPFILE
/tmp/.sops506055069/tmp-file291138648
sh-3.2$ cat $TMPFILE
{
        "database_password": "jf48t9wfw094gf4nhdf023r",
        "AWS_ACCESS_KEY_ID": "AKIAIOSFODNN7EXAMPLE",
        "AWS_SECRET_KEY": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}
sh-3.2$ ./program --config $TMPFILE
sh-3.2$ exit

# try to open the temporary file from earlier
$ cat /tmp/.sops506055069/tmp-file291138648
cat: /tmp/.sops506055069/tmp-file291138648: No such file or directory
```

Additionally, on unix-like platforms, both `exec-env` and `exec-file`
support dropping privileges before executing the new program via the
`--user <username>` flag. This is particularly useful in cases where the
encrypted file is only readable by root, but the target program does not
need root privileges to function. This flag should be used where
possible for added security.

To overwrite the default file name (`tmp-file`) in `exec-file` use the
`--filename <filename>` parameter.

``` sh
# the encrypted file can't be read by the current user
$ cat out.json
cat: out.json: Permission denied

# execute sops as root, decrypt secrets, then drop privileges
$ sudo sops exec-env --user nobody out.json 'sh'
sh-3.2$ echo $database_password
jf48t9wfw094gf4nhdf023r

# dropped privileges, still can't load the original file
sh-3.2$ id
uid=4294967294(nobody) gid=4294967294(nobody) groups=4294967294(nobody)
sh-3.2$ cat out.json
cat: out.json: Permission denied
```

# AWS CLI

!!! note
    Now AWS CLI was installed to the ABCI system. You can use the ``aws`` command simply by executing ``module load aws-cli``.

## Overview

This page describes installation of AWS command line interface (awscli below) and command examples.

## Installation of awscli

```
[username@es1 testdir]$ pip install awscli
```

## Register access token

Register your AWS access token.
```
[username@es1 testdir]$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]:
```

## command example

* Creates an S3 bucket
```
[username@es1 testdir]$ aws s3 mb s3://abci-access-test
make_bucket: abci-access-test
```

* Copy a local file to S3 bucket (cp)
```
[username@es1 testdir]$ ls -la 1gb.dat 
-rw-r--r-- 1 username grpname 1073741824 Nov  7 11:27 1gb.dat
[username@es1 testdir]$ aws s3 cp 1gb.dat s3://abci-access-test
upload: ./1gb.dat to s3://abci-access-test/1gb.dat
```

* List S3 object in the bucket (ls) 
```
[username@es1 testdir]$ aws s3 ls s3://abci-access-test 
2018-11-09 10:13:56 1073741824 1gb.dat
```

* Delete S3 object in the bucket (rm)
```
[username@es1 testdir]$ aws s3 rm s3://abci-access-test/1gb.dat
delete: s3://abci-access-test/1gb.dat
[username@es1 testdir]$ ls -l dir-test/
total 2097152

-rw-r--r-- 1 username grpname   1073741824 Nov  9 10:16 1gb.dat.1
-rw-r--r-- 1 username grpname   1073741824 Nov  9 10:17 1gb.dat.2
```

* Sync and recursively copy local file to bucket(sync)
```
[username@es1 testdir]$ aws s3 sync dir-test s3://abci-access-test/dir-test
upload: dir-test/1gb.dat.2 to s3://abci-access-test/dir-test/1gb.dat.2
upload: dir-test/1gb.dat.1 to s3://abci-access-test/dir-test/1gb.dat.1
```

* Sync and recursively copy file to bucket(sync)
```
[username@es1 testdir]$ aws s3 sync s3://abci-access-test/dir-test s3://abci-access-test/dir-test2
copy: s3://abci-access-test/dir-test/1gb.dat.1 to s3://abci-access-test/dir-test2/1gb.dat.1
copy: s3://abci-access-test/dir-test/1gb.dat.2 to s3://abci-access-test/dir-test2/1gb.dat.2
[username@es1 testdir]$ aws s3 ls s3://abci-access-test/dir-test2/
2018-11-09 10:20:05 1073741824 1gb.dat.1
2018-11-09 10:20:06 1073741824 1gb.dat.2
```

* Sync directories and recursively copy file to local directory (sync)
```
[username@es1 testdir]$ aws s3 sync s3://abci-access-test/dir-test2 dir-test2
download: s3://abci-access-test/dir-test2/1gb.dat.2 to dir-test2/1gb.dat.2
download: s3://abci-access-test/dir-test2/1gb.dat.1 to dir-test2/1gb.dat.1
[username@es1 testdir]$ ls -l dir-test2
total 2097152
-rw-r--r-- 1 username grpname 1073741824 Nov  9 10:20 1gb.dat.1
-rw-r--r-- 1 username grpname 1073741824 Nov  9 10:20 1gb.dat.2
```

* Deletes an S3 object in the bucket
```
[username@es1 testdir]$ aws s3 rm --recursive s3://abci-access-test/dir-test
delete: s3://abci-access-test/dir-test/1gb.dat.2
delete: s3://abci-access-test/dir-test/1gb.dat.1
[username@es1 testdir]$ aws s3 rm --recursive s3://abci-access-test/dir-test2
delete: s3://abci-access-test/dir-test2/1gb.dat.2
delete: s3://abci-access-test/dir-test2/1gb.dat.1
```

* Deletes an empty S3 bucket. 
```
[username@es1 testdir]$ aws s3 rb s3://abci-access-test
remove_bucket: abci-access-test
```

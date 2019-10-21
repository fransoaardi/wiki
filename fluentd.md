# install 

## ruby install

### using rvm

#### rvm install

```bash
$ yum install gcc-c++ patch readline readline-devel zlib zlib-devel \
   libyaml-devel libffi-devel openssl-devel make \
   bzip2 autoconf automake libtool bison iconv-devel sqlite-devel

$ curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
$ curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -

$ curl -L get.rvm.io | bash -s stable

$ source /home/deploy/.rvm/scripts/rvm

$ rvm reload

$ rvm requirements run

$ rvm install 2.6

$ rvm use 2.6 --default
```

#### reference: 
  - https://tecadmin.net/install-ruby-latest-stable-centos/
  - https://docs.fluentd.org/installation/install-by-gem

### build from source

```
$ curl -O https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.4.tar.gz
$ tar -xvf ruby-2.6.4.tar.gz
$ cd ruby-2.6.4
$ ./configure
$ make
$ sudo make install
```

#### reference:
  - https://www.ruby-lang.org/ko/documentation/installation/


## fluentd install

### install fluentd using gem
```bash
$ gem install fluentd --no-document
```

## fluentd test
```bash
$ fluentd --setup ./fluent
$ fluentd -c ./fluent/fluent.conf -v &

# using different terminal
$ echo '{"json":"message"}' | fluent-cat debug.test
```

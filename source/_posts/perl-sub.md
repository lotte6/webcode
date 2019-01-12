---
title: perl 引用
categories: tech
tag: hide
date: 2018-12-24 11:30:56
tags:
---
```
#!/usr/bin/perl -w
use strict;
my $sub = sub{
        my $server_name = shift;
        my $par = shift;
        print "server name is $server_name\n";
        print "$par->{\n";
        my $telnet = sub{ip}\n";
        print "$par->{port}
                print "we can type telnet $par->{ip} $par->{port}\n";
        }
};
$sub->('mysql',{'ip'=>'111.1.44.43','port' => '3306'});
$telnet->();
```
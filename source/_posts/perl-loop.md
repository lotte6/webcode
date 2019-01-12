---
title: perl 循环
categories: tech
tag: hide
date: 2018-12-24 11:28:55
tags:
---
```
#!/usr/bin/perl -w
use strict;
$" = ',';
my $min = 0;
my @list;
my @letter = (1 .. 33);
for(@letter){
push(@list,$_);
if(@list == 100){
print "@list[0 .. 99],\n";
@list = ();
  $min+=100;
}
}
print "this is min : $min\n";
print "@letter[$min .. $#letter]\n";
```
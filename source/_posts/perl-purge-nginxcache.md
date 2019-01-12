---
title: perl_purge_nginxcache
categories: tech
tag: hide
date: 2018-12-24 11:32:07
tags:
---
```
#!/usr/bin/perl -w
use strict;
use DBI;
use LWP::UserAgent;

my $browser = sub{
my $ua = LWP::UserAgent->new;

  #$ua->agent("");
  # tCreate a request
  my $req = HTTP::Request->new(GET =>shift);
  $req->content_type('application/x-www-form-urlencoded');
  $req->content('query=libwww-perl&mode=dist');
  # Pass request to the user agent and get a response back
  my $res = $ua->request($req);
  # Check the outcome of the response


unless($res->is_success) {


      print $res->status_line, "\n";
  }
};


my $result;
my $db = DBI->connect('DBI:mysql:database=novel;host=121.18.211.87',"read","readnovel");
#$db->prepare("Select count(*) co From novel.pa_articletext_info where articleid in (Select articleid From novel.pa_article where lastupdate>unix_timestamp()-60*15 order by lastupdate desc)")
my $sth = $db->prepare("select a.articleid,count(a.articleid) num from pa_article a,pa_articletext_info b  where  a.lastupdate>unix_timestamp()-60*15 and a.articleid = b.articleid group by a.articleid");
$sth->execute();
while($result = $sth->fetchrow_hashref()){
        my $id=substr ("$result->{articleid}",-1,1);
        for(my $i = 1;$i <= $result->{num};$i++){
#                $browser->("http://127.0.0.1/purge/novel/$result->{articleid}/$i.html");
                unlink "/var/www/readnovel/novel$id/$result->{articleid}/$i.html";
        }
}
$sth->finish;
$db->disconnect;
```
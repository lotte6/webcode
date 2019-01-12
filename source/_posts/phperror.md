---
title: 'make: *** [sapi/cli/php] Error 1'
categories: tech
tag: hide
date: 2018-12-13 19:16:23
tags:
---

```
ext/xmlreader/.libs/php_xmlreader.o(.text+0x126e): In function `zim_xmlreader_setSchema':
/back/soft/php-5.2.6/ext/xmlreader/php_xmlreader.c:983: undefined reference to `xmlTextReaderSchemaValidate'
ext/xmlreader/.libs/php_xmlreader.o(.text+0x14f5): In function `zim_xmlreader_XML':
/back/soft/php-5.2.6/ext/xmlreader/php_xmlreader.c:1109: undefined reference to `xmlTextReaderSetup'
collect2: ld returned 1 exit status
make: *** [sapi/cli/php] Error 1
```

解决办法：
make ZEND_EXTRA_LIBS='-liconv'

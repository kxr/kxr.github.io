---
layout: post
title: Migrating your squirrel mail contacts to roundcube
---

A simple script that will convert squirrel web mail contacts of each user to vcard format that can then be imported to roundcube webmail.

{% highlight bash %}
#!/bin/bash

for x in `ls *.abook`
do
    cat $x | while read line
    do
        echo "BEGIN:VCARD" &gt;&gt; $x.vcf
        echo "VERSION:3.0" &gt;&gt; $x.vcf
        echo -n "N:" &gt;&gt; $x.vcf
        echo -n `echo $line | cut -d'|' -f3` &gt;&gt; $x.vcf
        echo -n ";" &gt;&gt; $x.vcf
        echo `echo $line | cut -d'|' -f2` &gt;&gt; $x.vcf
        echo -n "FN:" &gt;&gt; $x.vcf
        echo `echo $line | cut -d'|' -f1` &gt;&gt; $x.vcf
        echo -n "EMAIL;TYPE=PREF,INTERNET:" &gt;&gt; $x.vcf
        echo `echo $line | cut -d'|' -f4` &gt;&gt; $x.vcf
        echo "END:VCARD" &gt;&gt; $x.vcf
        echo &gt;&gt; $x.vcf
    done
    echo "done $x"
done
{% endhighlight %}

To use this script: login on the server where squirrel mail is installed and browse to squirrel mail's data directory where squirrel mail keeps its .abook and .pref and .sig files.

Next create a file by the name of "makevcard" and paste this script in that file.

Next make the file executeable by running the following command

> chmod +x ./makevcard

Now execute the script by

> ./makevcard

Squirrel mail saves the address book for every user in a separate file, for example name@domain.com.abook. After this script is executed, For every .abook file, you'll get .abook.vcf file that will be in vcard format.


copy all the .vcf files to any location you like and you can now use your roundcube built in "Import Contacts" Feature to import contacts of individual user;

---
id: 89
title: Generate messages.properties and messages_xx.properties from CSV
date: 2013-04-26T20:57:41+00:00
author: Björn Bilger
permalink: /2013/04/26/generate-messages-properties-and-messages_xx-properties-from-csv/
categories:
  - Java
  - Java EE
  - JSF
tags:
  - automation
  - constant class
  - constants
  - Eclipse Builder
  - Excel
  - generate
  - generation
  - I18N
  - LibreOffice
  - localization
  - messages.properties
  - Perl
  - spreadsheet
---
# Intro

I am currently working on web project, which is localized and I lost the overview over which messages need to be translated. So I thought that it would be better to manage the messages in a spreadsheet rather than having the messages along with the keys in different files. So I wrote a small perl script that parses the spreadsheet and generates the messages_xx.properties files for me. The script gets invoked automatically when I build the project in Eclipse. (You can, however, invoke the script from your build scripts, too, if you like.)

For me this is the perfect solution, because I always have a good overview about all message keys that still need a translation.

Another advantage of this solution is that the script also generates a class with the message keys as constants. So you don't need to use raw strings, anymore in your code.<!--more-->

To make sure that we are on the same page, here's an example:

#### Input

##### messages.csv

[<img width="230" height="86" alt="table" src="{{ site.baseurl }}/images/posts/2013/04/table.png"/>]({{ site.baseurl }}/images/posts/2013/04/table.png)

or rather

```
"key";"en";"de"
"hello";"Hello";"Hallo"
"world";"World";"Welt"
```

#### Output

##### messages.properties

```
# auto-generated on Fri Apr 26 20:42:01 2013

hello=Hello
world=World
```

##### messages_de.properties

```
# auto-generated on Fri Apr 26 20:42:01 2013

hello=Hallo
world=Welt
```

##### Messages.java

``` java
package org.bbilger.util;

// auto-generated on Fri Apr 26 20:42:01 2013
public final class Messages {
  public final static String HELLO = "hello";
  public final static String WORLD = "world";
}
```

# Setup

Since it is a perl script, you need to install perl. In addition, you must install an additional package: `Text::CSV\_XS`. On Linux, you can simply run the program `cpan` and install the package by executing the command `install Text::CSV\_XS`.

I'll give you a quick example on how to set up everything but you need to adapt it to your needs, of course. (you can find some [screenshots](#screenshots "screenshots") at the end of the article)

  1. Create a new Maven project called `MessagesBuilder`
  2. Save the [perl script below](#script "perl script below") as `messages_builder.pl` under `build` in your Eclipse project.
  3. Create a class named **Messages** in the package com.bbilger.util. (you want to adapt this obviously 🙂 )
  4. Create a new file called `messages.csv`; in your resource folder .
  5. Open `messages.csv` with LibreOffice. Choose **Unicode (UTF-8)** for encoding, **;** as field delimiter and a double-quote as text delimiter
  6. Write **Keys** in the first line in the first column. In the first line of the following columns, you must write the language codes. In the example, I used **en** in the second and **de** in the third column. So it's obvious that your message keys then go into the first column and your localized messages into the following columns.
  7. In order to export the spreadsheet properly, perform the following: **File->Save As...**. Set the name to `messages.csv`, make sure you check edit filter settings and confirm that you want to export the file as csv. In the upcoming **Export Text File** dialog, make sure to change the field delimiter to **;** and check the option **Quote all text cells**.
  8. Now we can set up the builder in Eclipse. Select the project and perform Project->Properties. Select Builders in the upcoming dialog, click on **New...** and select Program. In the **Main** tab of the builder configuration dialog, you have to:
  9. Set the name to **Messages Builder** (or whatever you like)
 10. Set the location to `${system_path:perl}`
 11. Set the arguments: `${workspace\_loc:/MessagesBuilder/build/messages\_builder.pl}` `${workspace\_loc:/MessagesBuilder/src/main/resources}` `${workspace\_loc:/MessagesBuilder/src/main/java/org/bbilger/util/Messages.java}` **Messages** `org.bbilger.util`
 12. In the **Build options** tab, you should check **During a Clean**

If you perform a clean now, two new files should be created under resources: `messages.properties` and `messages_de.properties`. In addition, the class **Messages** should be updated and will contain all message key strings as constants. The names of the constants is equivalent to the key, but they are in uppercase.

Obviously you have to perform the steps 5 to 7 whenever you want to add a new message.

Note: Using Excel instead of LibreOffice, is possible, too, of course.

# Script

Note that the script is a quick and dirty solution for my problem so it contains the absolute minimum, only. It's neither very fault tolerant nor very beautiful, but it is to be used by developers anyways 😉

The script expects the following arguments:

  1. The path of the resource directory. It expects a file called `messages.csv` to be there and the files `messages.properties` and `messages_xx.properties` will be created into that directory, too.
  2. The path to the messages class.
  3. The name of the messages class.
  4. The package name for the messages class.

Expectations, requirements and recommendations:

  1. The spreadsheet file must be `messages.csv` and it must be located in the same directory, the property files are generated to.
  2. The generated files will be message.properties for the en (=default) localization and the other ones will be messages_xx.properties (xx is the language code given as column header).
  3. The keys should be given with this format `word1_word2` because they will transformed into uppercase and used as constants in the `Messages` class.
  4. The script expects the fields to be delimited by semicolon.
  5. The script expects `messages.csv` to be UTF-8 encoded.

What is the script doing:

  1. Parses `messages.csv` (expects UTF-8 encoding) and creates an internal representation of it. The internal representation is a list of arrays. Whereas the list represents the lines and the arrays represent the fields.
  2. Overwrites the files `messages.properties` and `messages_xx.properties`. These will contain the keys and the appropriate localized message in the format `key=locaizedMessage`. In case no localized message is provided, a comment is inserted into the properties file: `# key`. The encoding of these files is **ISO 8859-1** or rather **Latin1**, which is equivalent. This is required by Java: <http://docs.oracle.com/javase/6/docs/api/java/util/Properties.html>
  3. Generates message.properties and messages_xx.properties and the messages class.

``` perl
use strict;
use warnings;
use Text::CSV_XS;

if (@ARGV != 4) {
  die "Usage: perl messages_builder.pl    \n".
    "perl messages_builder.pl '/somedir/src/main/resources' '/somedir/src/main/java/com/mypackage/Messages.java' 'Messages' 'com.mypackage'\n";
}

my $messages_dir = $ARGV[0];
my $messages_class_path = $ARGV[1];
my $messages_class_name = $ARGV[2];
my $messages_class_package = $ARGV[3];

my $csv_messages = $messages_dir."/messages.csv";
my $csv = Text::CSV_XS-&gt;new ({ binary =&gt; 1, sep_char =&gt; ';', quote_char =&gt; '"' }) or
     die "Cannot use CSV: ".Text::CSV_XS-&gt;error_diag ();

open my $fh, "&lt;", $csv_messages or die "$csv_messages: $!"; my $header_line = $csv-&gt;getline ($fh);
my @headers = ();
my @properties_lists = ();
my @keys = ();

foreach my $header (@$header_line) {
  if ($header ne "") {
    push(@headers, $header);
    push(@properties_lists, []);
  }
}

while (my $line = $csv-&gt;getline ($fh)) {
  my @fields = @$line;

  for (my $i = 0; $i &lt; @fields && $i &lt; @headers; $i++) {
    my $field = $fields[$i];
    if ($i == 0) {
      push(@keys, $field);
    } else {
      push($properties_lists[$i], $field);
    }
  }
}

close $fh;

for (my $i = 1; $i &lt; @headers; $i++) {   my $header = $headers[$i];   my $properties_list = $properties_lists[$i];   my $prop_file = $messages_dir."/messages";   if ($header ne "en") {     $prop_file .= "_".$header;   }   $prop_file .= ".properties";   open $fh, "&gt; :encoding(Latin1)", $prop_file or die "$prop_file: $!";
  binmode($fh, ":encoding(Latin1)");
  print $fh "# auto-generated on ".localtime."\n\n";
  for (my $j = 0; $j &lt; @keys; $j++) {
    my $message = "";
    if ($j &lt; @$properties_list) {             $message = $$properties_list[$j];     }     if ($message eq "") {        printf $fh "# ";     }     print $fh $keys[$j]."=".$message."\n";   }   close $fh; } open $fh, "&gt; :encoding(utf8)", $messages_class_path or die "$messages_class_path: $!";
binmode($fh, ":encoding(utf8)");
print $fh "package ".$messages_class_package.";\n\n";
print $fh "// auto-generated on ".localtime."\n";
print $fh "public final class ".$messages_class_name." {\n";
for my $key (@keys) {
  print $fh "\tpublic final static String ".(uc $key)." = \"".$key."\";\n";
}
print $fh "}";
```

# Screenshots {#screenshots}

## LibreOffice CSV Export Settings

[<img width="300" height="129" alt="Libre Office CSV Export" src="{{ site.baseurl }}/images/posts/2013/04/libre_office_csv_export.png"/>]({{ site.baseurl }}/images/posts/2013/04/libre_office_csv_export.png)

## Project Structure

[<img width="188" height="300" alt="project structure" src="{{ site.baseurl }}/images/posts/2013/04/project_structure.png"/>]({{ site.baseurl }}/images/posts/2013/04/project_structure.png)

## Table

[<img width="230" height="86" alt="table" src="{{ site.baseurl }}/images/posts/2013/04/table.png"/>]({{ site.baseurl }}/images/posts/2013/04/table.png)

## Builder Settings

[<img width="300" height="250" alt="Eclipse builder settings 2" src="{{ site.baseurl }}/images/posts/2013/04/eclipse_builder_settings_2.png"/>]({{ site.baseurl }}/images/posts/2013/04/eclipse_builder_settings_2.png)

[<img width="300" height="250" alt="Eclipse builder settings" src="{{ site.baseurl }}/images/posts/2013/04/eclipse_builder_settings.png"/>]({{ site.baseurl }}/images/posts/2013/04/eclipse_builder_settings.png)

# Final Words

I hope this helps 😉

---
id: 89
title: Generate messages.properties and messages_xx.properties from CSV
date: 2013-04-26T20:57:41+00:00
author: Björn Bilger
layout: post
guid: http://yabapal.wordpress.com/?p=89
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

Another advantage of this solution is that the script also generates a class with the message keys as constants. So you don&#8217;t need to use raw strings, anymore in your code.<!--more-->

To make sure that we are on the same page, here&#8217;s an example:

#### Input

##### messages.csv

[<img class="alignnone size-full wp-image-96" alt="table" src="http://i0.wp.com/yabapal.files.wordpress.com/2013/04/table.png?resize=230%2C86" data-recalc-dims="1" />](http://i0.wp.com/yabapal.files.wordpress.com/2013/04/table.png)

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

Since it is a perl script, you need to install perl. In addition, you must install an additional package: &#8220;Text::CSV\_XS&#8221;. On Linux, you can simply run the program &#8220;cpan&#8221; and install the package by executing the command &#8220;install Text::CSV\_XS&#8221;.

I&#8217;ll give you a quick example on how to set up everything but you need to adapt it to your needs, of course. (you can find some [screenshots](#screenshots "screenshots") at the end of the article)

  1. Create a new Maven project called &#8220;MessagesBuilder&#8221;
  2. Save the [perl script below](#script "perl script below") as &#8220;messages_builder.pl&#8221; under &#8220;build&#8221; in your Eclipse project.
  3. Create a class named &#8220;Messages&#8221; in the package com.bbilger.util. (you want to adapt this obviously 🙂 )
  4. Create a new file called &#8220;messages.csv&#8221; in your resource folder .
  5. Open &#8220;messages.csv&#8221; with LibreOffice. Choose &#8220;Unicode (UTF-8)&#8221; for encoding, &#8220;;&#8221; as field delimiter and a double-quote as text delimiter
  6. Write &#8220;Keys&#8221; in the first line in the first column. In the first line of the following columns, you must write the language codes. In the example, I used &#8220;en&#8221; in the second and &#8220;de&#8221; in the third column. So it&#8217;s obvious that your message keys then go into the first column and your localized messages into the following columns.
  7. In order to export the spreadsheet properly, perform the following: File->&#8221;Save As&#8230;&#8221;. Set the name to &#8220;messages.csv&#8221;, make sure you check edit filter settings and confirm that you want to export the file as csv. In the upcoming &#8220;Export Text File&#8221; dialog, make sure to change the field delimiter to &#8220;;&#8221; and check the option &#8220;Quote all text cells&#8221;.
  8. Now we can set up the builder in Eclipse. Select the project and perform Project->Properties. Select Builders in the upcoming dialog, click on &#8220;New&#8230;&#8221; and select Program. In the &#8220;Main&#8221; tab of the builder configuration dialog, you have to:
  9. Set the name to &#8220;Messages Builder&#8221; (or whatever you like)
 10. Set the location to &#8220;${system_path:perl}&#8221;
 11. Set the arguments: ${workspace\_loc:/MessagesBuilder/build/messages\_builder.pl} &#8220;${workspace\_loc:/MessagesBuilder/src/main/resources}&#8221; &#8220;${workspace\_loc:/MessagesBuilder/src/main/java/org/bbilger/util/Messages.java}&#8221; &#8220;Messages&#8221; &#8220;org.bbilger.util&#8221;
 12. In the &#8220;Build options&#8221; tab, you should check &#8220;During a Clean&#8221;

If you perform a clean now, two new files should be created under resources: &#8220;messages.properties&#8221; and &#8220;messages_de.properties&#8221;. In addition, the class &#8220;Messages&#8221; should be updated and will contain all message key strings as constants. The names of the constants is equivalent to the key, but they are in uppercase.

Obviously you have to perform the steps 5 to 7 whenever you want to add a new message.

Note: Using Excel instead of LibreOffice, is possible, too, of course.

# Script

Note that the script is a quick and dirty solution for my problem so it contains the absolute minimum, only. It&#8217;s neither very fault tolerant nor very beautiful, but it is to be used by developers anyways 😉

The script expects the following arguments:

  1. <span style="line-height:13px;">The path of the resource directory. It expects a file called &#8220;messages.csv&#8221; to be there and the files &#8220;messages.properties&#8221; and &#8220;messages_xx.properties&#8221; will be created into that directory, too.</span>
  2. The path to the messages class.
  3. The name of the messages class.
  4. The package name for the messages class.

Expectations, requirements and recommendations:

  1. <span style="line-height:13px;">The spreadsheet file must be &#8220;messages.csv&#8221; and it must be located in the same directory, the property files are generated to.</span>
  2. The generated files will be message.properties for the en (=default) localization and the other ones will be messages_xx.properties (xx is the language code given as column header).
  3. The keys should be given with this format &#8220;word1_word2&#8221; because they will transformed into uppercase and used as constants in the &#8220;Messages&#8221; class.
  4. The script expects the fields to be delimited by semicolon.
  5. The script expects &#8220;messages.csv&#8221; to be UTF-8 encoded.

What is the script doing:

  1. <span style="line-height:13px;">Parses &#8220;messages.csv&#8221; (expects UTF-8 encoding) and creates an internal representation of it. The internal representation is a list of arrays. Whereas the list represents the lines and the arrays represent the fields.</span>
  2. Overwrites the files &#8220;messages.properties&#8221; and &#8220;messages_xx.properties&#8221;. These will contain the keys and the appropriate localized message in the format &#8220;key=locaizedMessage&#8221;. In case no localized message is provided, a comment is inserted into the properties file: &#8220;# key&#8221;. The encoding of these files is &#8220;ISO 8859-1&#8221; or rather &#8220;Latin1&#8221;, which is equivalent. This is required by Java: <http://docs.oracle.com/javase/6/docs/api/java/util/Properties.html>
  3. Generates message.properties and messages_xx.properties and the messages class.

<span id="script"> </span>

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

for (my $i = 1; $i &lt; @headers; $i++) { 	my $header = $headers[$i]; 	my $properties_list = $properties_lists[$i]; 	my $prop_file = $messages_dir."/messages"; 	if ($header ne "en") { 		$prop_file .= "_".$header; 	} 	$prop_file .= ".properties"; 	open $fh, "&gt; :encoding(Latin1)", $prop_file or die "$prop_file: $!";
	binmode($fh, ":encoding(Latin1)");
	print $fh "# auto-generated on ".localtime."\n\n";
	for (my $j = 0; $j &lt; @keys; $j++) {
		my $message = "";
		if ($j &lt; @$properties_list) {			 			$message = $$properties_list[$j]; 		} 		if ($message eq "") { 			 printf $fh "# "; 		} 		print $fh $keys[$j]."=".$message."\n"; 	} 	close $fh; } open $fh, "&gt; :encoding(utf8)", $messages_class_path or die "$messages_class_path: $!";
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

[<img class="alignnone size-medium wp-image-94" alt="libre_office_csv_export" src="http://i2.wp.com/yabapal.files.wordpress.com/2013/04/libre_office_csv_export.png?resize=300%2C129" data-recalc-dims="1" />](http://i2.wp.com/yabapal.files.wordpress.com/2013/04/libre_office_csv_export.png)

## Project Structure

[<img class="alignnone size-medium wp-image-95" alt="project_structure" src="http://i1.wp.com/yabapal.files.wordpress.com/2013/04/project_structure.png?resize=188%2C300" data-recalc-dims="1" />](http://i1.wp.com/yabapal.files.wordpress.com/2013/04/project_structure.png)

## Table

[<img class="alignnone size-full wp-image-96" alt="table" src="http://i0.wp.com/yabapal.files.wordpress.com/2013/04/table.png?resize=230%2C86" data-recalc-dims="1" />](http://i0.wp.com/yabapal.files.wordpress.com/2013/04/table.png)

## Builder Settings

[<img class="alignnone size-medium wp-image-102" alt="eclipse_builder_settings_2" src="http://i0.wp.com/yabapal.files.wordpress.com/2013/04/eclipse_builder_settings_2.png?resize=300%2C250" data-recalc-dims="1" />](http://i0.wp.com/yabapal.files.wordpress.com/2013/04/eclipse_builder_settings_2.png)

[<img class="alignnone size-medium wp-image-93" alt="eclipse_builder_settings" src="http://i1.wp.com/yabapal.files.wordpress.com/2013/04/eclipse_builder_settings.png?resize=300%2C142" data-recalc-dims="1" />](http://i1.wp.com/yabapal.files.wordpress.com/2013/04/eclipse_builder_settings.png)

# Final Words

I hope this helps 😉
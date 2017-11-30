# OverTheWire-Natas
Walk-through guide with examples for the OverTheWire Natas game

## Level 00
**Needs:** web browser

User/Pass: `natas0`

URL: http://natasX.natas.labs.overthewire.org where X is the level

Connect to page and view page source
* Password found in source

## Level 01
Use Ctrl+U to open page source since right click disabled

## Level 02
Check page source and notice the `/files/pixel.png` image

Check out http://...../files
* Open `users.txt`
  * Password here

## Level 03
No leaking of info in page source

Look into `robots.txt`
* For web crawlers

`Robots.txt` - informs robot (web crawler) about which areas of the website should not be processed or scanned

`Robots.txt`
* User-agent: \*
* Disallow: `/s3cr3t/`

`..../s3cr3t/` -> has `users.txt` file
* Password!!!!

## Level 04
Use Burp Suite
* Examine referer header

Change referer header to point from `natas5`
* Password granted

## Level 05
Even after passing the correct password, response is that I am "Not registered for access"

Using OWASP Zap, examining packets being passed on login to `natas5`
* Attempt: Change of `wfid` from `wfid=6` to `5`

Using Burp Suite (by Portswigger) to capture traffic between natas5 and you
* Look for `loggedin` property for cookie and set to 1
  * Password granted

## Level 06
After passing User/Pass for `natas6`, presented with `Input secret:` request to obtain `natas7` password
* Has link that shows source code for page

Go to page: http://natas6....org/includes/secret.inc
* Examine webpage source code to get secret
  * FOEIUWGHFEEUHOFUOIU
  * Input into query => Password granted

## Level 07
Page with two links: 'Home' and 'About'

Examining source code for the `natas7` page
* 'Home' - points to 'index.php?page=home'
* 'About' - points to 'index.php?page=about'
* Hint embedded as commented code within the page source code
  * 'hint: password for webuser natas8 is in /etc/natas_webpass/natas8'

Home - displays 'this is the front page'
About - displays 'this is the about page'

Set web address to 'http://natas7.../index.php?page=/etc/natas_webpass_natas8'
* Got password!!!

## Level 08
Page displays an "Input secret" box + 'Submit Query' button and 'View source code' link

'View sourcecode' - href="index_source.html"

```
<?
$encodedSecret="3d3d516343746d4d6d6c315669563362"
function encodeSecret($secret) {
	return bin2hex(strrev(base64_encode($secret)));
}
if (array_key_exists("submit", $_POST)) {
	if (encodeSecret($_POST['secret'])==$encodedSecret) {
		print "Access granted. The password for natas9 is <censored>";
	} else {
		print "Wrong secret";
	}
}
?>
```

Learn what `bin2hex()`, `strrev()`, and `base64_encoded()` can do
* `base64_encode(void \*data, size_t size);
  * Function used to recode data to/from various standards to MIME data transfer encoding
  * `base64_decode(char \*string, size_t \*size)`
* `string strrev(string $string)`
  * Returns string, reversed
* `string bin2hex(string $str)`
  * Returns an ASCII string containing the hexadecimal representation of str.  The conversion is done byte-wise with high-nibble first.

Take `$encoded secret` and:
1. Use `hex2bin()` - `string hex2bin(string $data)`
  * Obtain binary string
2. Use `strrev()` to re-reverse string
3. Use `base64_decode()` to obtain original 'Input secret' input

Use sandbox.onlinephpfunctions.com to test PHP code online

`$testOutput = base64_decode(strrev(hex2bin($encodedSecret)));`
* Prints out as 'oubWYf2kBq'
  * Gets password!!!

## Level 09
Page with 'Find words containing' field with 'search' button, 'Output:' text area and 'view sourcecode' link

'View sourcecode' - `href="index-source.html"`

```
<pre>
<?
$key = "";
if (array_key_exists("needle", $_REQUEST)) {
	$key = $_REQUEST["needle"];
}
if ($key != "") {
	passthru("grep -i $key dictionary.txt");
}
?>
</pre>
```

grep `-i` flag is the `--ignore-case`
* Ignore case distinctions between PATTERN and input file

`bool array_key_exists(mixed $key, array $array)`
* check if the given key or index exists in the array

`$_REQUEST` - HTTP request variables
* An associative array that by default contains the contents of `$_GET`, `$_POST`, and `$_COOKIE`

If pass `a*z` returns large list in almost alphabetical order (ex: starts with 'Nazi' instead of 'ablaze'

**Note/Lead:** Use regex expressions to explore the contents of 'dictionary.txt'

Input: '; cat /etc/natas_webpass/natas10; ls'
* Get password!!!

## Level 10
Similar main page as Level 09, **but** with additional 'For security reasons, we now filter on certain characters'

Change in page code:
```
if ($key != "") {
	if (preg_match('/[;|]/', $key)) {
		print "Input contains an illegal character!";
	} else {
		passthru("grep -i $key dictionary.txt");
	}
}
```

`int preg_match(string $pattern, string $subject [, array &&matches [, in $flags = 0 [, int $offset = 0]]])` - Perform regular expression match
* Searches 'subject' for a match to the regular expression given in 'pattern'

Code searching for: `[ ; | & ]`

Try `\n` to separate commands

Input: `-v "#" -r /etc/natas_webpass/natas11`
* Got password!!!

## Level 11
Page says 'Cookies are protected with XOR encryption'.  Also has 'Background color' field with '#ffffff' filled in and a 'set color' button.  There is a 'view sourcecode' link

```
<?
$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");
function xor_encrypt($in) {
	$key = '<censored>';
	$text = $in;
	$outText = '';
	//Iterate through each character
	for ($i = 0; $i < strlen($text); $i++) {
		$outText .= $text[$i]^$key[$i % strlen($key)];
	}
	return $outText;
}
function loadData($def) {
	global $_COOKIE;
	$mydata = $def;
	if (array_key_exists("data", $_COOKIE)) {
		$tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
		if (is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
			if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
				$mydata['showpassword'] = $tempdata['showpassword'];
				$mydata['bgcolor'] = $tempdata['bgcolor'];
			}
		}
	}
	return $mydata;
}
function saveData($d) {
	setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
$data = loadData($defaultdata);
if (array_key_exists("bgcolor", $_REQUEST)) {
	if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
		$data['bgcolor'] = $_REQUEST['bgcolor'];
	}
}
saveData($data);
?>
```

`mixed json_decode(string $json [, bool $assoc = false [, int $depth = 512 [, int $options = 0]]])` - Decodes json string
* Takes a JSON encoded string and converts it into a PHP variable

`bool setcookie (string $name [, string $value [, int $expire = 0[, string $path [, string $domain [, bool $secure = false [, bool $httponly = false]]]]]])` - Send a cookie
* `setcookie()` defines a cookie to be sent along with the rest fo the HTTP headers
  * Once the cookies have been set, they can be accessed on the next page load with the `$_COOKIE` or `$HTTP_COOKIE_VARS`

`bool is_array(mixed $var)` - Finds whether a variable is an array
* Find whether the given variable is an array

'/^#(?:[a-f\d]{6})$/i'
1. Start of line must be '#' (comes from '^#')
2. Look for something that matches without creating a capturing group (non-capturing group)[comes from (?:)]

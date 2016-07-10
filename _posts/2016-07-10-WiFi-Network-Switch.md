---
layout: post
title: Switch WiFi Networks Via Command Line On Multiple Machines (Linux / Windows) At Once
tags: wifi, network, ubuntu, windows10, bash, powershell, python, winrm
---

I use two laptops at home. One runs Ubuntu and the other runs Windows 10. Generally I use a wireless router to connect the machines. But, sometimes when my broadband is down or slow, I use my Android phone as a wireless router and use it's 3G connection to connect to the Internet.

The default method of switching networks is well known. On Ubuntu, I select the wireless network in Network Manager. On Windows, I do so using the network selection applet accessible from system tray.

Simple, right? Except that, there are a number of problems with this approach:

1. Laziness, impatience and dislike of UI's (unless they are well programmed).

2. Windows network selection applet is too slow. Sometimes, it takes 30 seconds or more to populate it with available networks.

3. Windows keeps forgetting DNS settings as I switch between wireless networks. So, I have to go to network settings, select the wifi connection and edit the DNS settings manually. (I don't know if it is the norm, but, when I go to network and sharing center in Windows 10, it just hangs before displaying the network connections.)

4. I run a proxy server on Windows machine which is used by Linux machine to access the Internet.

5. I run a web server on Linux which I generally access from browser on Windows.

6. There are samba shares on Linux which I access from Windows. When I switch networks, they become inaccessible until I change the Linux machine's ip in hosts file.

7. I use Synergy to share keyboard and touchpad of Linux machine with Windows. Needless to say, it goes haywire when networks are switched. It needs to be restarted on Linux to pick up the new ip address on the new network and Windows needs its hosts files updated with ip address of Linux machine to connect to the Synergy server.

8. Taking care of all the above manually is tedious and directly makes the problem #1 worse.

In the following text and code, I'll illustrate my approach of automating all of the above to be executed in a single step.


# Windows

 Windows part is a two step process. First, we establish WinRM session and upload powershell scripts. Second, we execute those scripts.


### Start WinRM Session, Upload Powershell Scripts, Execute Them

 On Windows machine, I have WinRM installed, enabled and configured. A python script does the job of uploading files and doing the WinRM stuff.
 Python script uses pywinrm module. It's PyPI page has excellent notes on WinRM configuration as well.

 The script uses pysftp module to upload files to Windows. (Although, it'll be a good exercise to eliminate its use by using the powershell command Set-Content to upload files to Windows.) For setting up openssh and sftp on Windows, [here](https://winscp.net/eng/docs/guide_windows_openssh_server) are step by step instructions.

 The script also uses redcmd module to parse the command line.
 

 The script has two subcommands:

 1. switchnet: switches Windows to the given profile
 2. edit_hosts: edits Windows hosts file with given ip address for the Linux machine (See last step of Linux section for why we need this.)

 When given a profile name with switchnet subcommand:

 1. it uploads relevant config file to Windows (using sftp, see details of config file in section for Linux)
 2. then uploads the network switching script itself to Windows
 3. starts a WinRM session using username/password (basic) authentication
 4. executes the script on Windows

 {% highlight bash %}
 $ python win10.py switchnet phone
 {% endhighlight %}

 {% highlight python %}
 import winrm
 from getpass import getpass
 import keyring
 import sys
 import pysftp
 from requests.exceptions import ReadTimeout
 from os.path import join as joinpath, realpath, dirname
 
 from redcmd.api import subcmd, execute_commandline, Arg
 
 
 class Win10:
 
 	def __init__(self):
 		self.hostname 	= 'win10'
 		self.username 	= 'username'
 		self.session 	= None
 		self.password 	= self.get_password()
 
 		self.remote_path = "C:\\Users\\username\\Documents\\sftp\\"
 		self.script_path = dirname(realpath(__file__))
 
 
 	def get_password(self):
 		passwd = keyring.get_password('system', 'win10')
 		if passwd is None:
 			passwd = getpass('enter password: ')
 			keyring.set_password('system', 'win10', passwd)
 
 		return passwd
 
 
 	def upload_files(self, file_list):
 		sftp_path = "Documents/sftp"
 
 		with pysftp.Connection(self.hostname, username=self.username, password=self.password) as sftp:
 			with sftp.cd(sftp_path):
 				for f in file_list:
 					sftp.put(joinpath(self.script_path, f))
 					print 'uploaded:', f
 
 
 
 	def run_cmd(self, cmd, args=None, ps=False):
 		if self.session is None:
 			self.session = winrm.Session(self.hostname, auth=(self.username, self.password))
 
 		if not ps:
 			return self.session.run_cmd(cmd, args)
 		else:
 			return self.session.run_ps(cmd)
 
 
 	def execute_uploaded_ps(self, script_name, *args):
 		r = self.run_cmd('powershell', args=[script_name] + list(args))
 		print r.std_out
 
 
 	def execute_ps(self, script_name, *args):
 		self.upload_files([script_name])
 		self.execute_uploaded_ps(self.remote_path + script_name, *args)
 
 
 @subcmd
 def switchnet(name=Arg(default='wrouter', choices=['wrouter', 'phone'])):
 	w10 = Win10()
 	w10.upload_files([name + '.cfg'])
 	try:
 		w10.execute_ps('switchnet.ps1', name)
 	except ReadTimeout as e:
 		print 'disconnected WinRM session from Windows'
 
 
 @subcmd
 def edit_hosts(ip):
 	w10 = Win10()
 	w10.execute_ps('edit_hosts.ps1', ip)
 
 
 if __name__ == '__main__':
 	execute_commandline(_to_hyphen=True)
 
 {% endhighlight %}


### Switch Wireless Network, Change DNS And Edit Hosts File
 
 This step is part of the previous step.

 A powershell script (named switchnet.ps1) does all the operations on Windows. As we saw in previous step, it is uploaded to Windows and executed via WinRM by a python script.

 The script uses netsh to switch network and update the DNS server. It uses config file as described in section for Linux to read in the wifi profile name, last known ip of Linux machine and DNS server ip.

 {% highlight powershell %}
 param (
 	[ValidateSet('wrouter', 'phone')][String]$name = "wrouter"
 )
 
 function Edit-HostsFile ($ip) {
 	[String]$hosts_file_path = "C:\Windows\System32\drivers\etc\hosts"
 	(Get-Content $hosts_file_path) -replace ".* ubuntu", "$ip ubuntu" | Set-Content $hosts_file_path
 }
 
 function Select-WifiNetwork ($profile) {
 	Invoke-Expression "netsh wlan connect $profile"
 }
 
 function Change-DNS ($conn, $dns) {
 	Invoke-Expression "netsh interface ip delete dnsservers $conn all"
 	Invoke-Expression "netsh interface ip add dns name=$conn $dns"
 }
 
 function Get-IP ($conn) {
 	netsh interface ip show ipaddresses $conn | Select-String "Address (.*) Parameters" | % {$_.matches[0].groups[1].value}
 }
 
 function Get-Config ($filename) {
 	Get-Content (Join-Path $PSScriptRoot $filename) | foreach-Object -begin {$h=@{}} -process { $k = [regex]::split($_,'='); if(($k[0].CompareTo("") -ne 0) -and ($k[0].StartsWith("[") -ne $True)) { $h.Add($k[0], $k[1]) } }
 	$h
 }
 
 
 function Main() {
 	[String]$conn = "Wi-Fi"
 
 	$config = Get-Config ($name + ".cfg")
 
 	$wifi_profile = $config.wifi_profile
 	$ubuntu_ip = $config.ubuntu_ip
 	$dns = $config.dns
 
 	Select-WifiNetwork $wifi_profile
 	Write-Output "selected wifi network: $wifi_profile"
 
 	Change-DNS $conn, $dns
 	Write-Output "change primary dns to: $dns"
 
 	Edit-HostsFile $ubuntu_ip
 	Write-Output "updated hosts file with ubuntu ip: $ubuntu_ip"
 
 	Write-Output "ip address: $(Get-IP $conn)"
 
 	exit 0
 }
 
 Main
 {% endhighlight %}


### Optionally, Update IP Address Of Linux Machine

 This is done from Linux, so, the details are in that section. Here, I'll just show the powershell script (named edit_hosts.ps1) which uploaded to Windows and executed by python script.

 {% highlight powershell %}
 param (
 	[String]$ip
 )
 
 function Edit-HostsFile ($ip) {
 	[String]$hosts_file_path = "C:\Windows\System32\drivers\etc\hosts"
 	(Get-Content $hosts_file_path) -replace ".* ubuntu", "$ip ubuntu" | Set-Content $hosts_file_path
 }
 
 Edit-HostsFile $ip
 {% endhighlight %}


### Synergy Client Restart

 It is not needed !! The synergy service picks up the updated ip address from hosts file and connects to it in a few seconds. This serendipitous behavior saves us some code.


# Linux

### Switch Wireless Network Using 'nmcli'

 {% highlight bash %}
 nmcli c up id <wifi_network_profile_name>
 {% endhighlight %}


### Update Hosts File With Windows IP Address

 This script receives profile name as first command line argument ($1). 
 Settings for a profile are stored in a config file in the same directory. The name of the config file is <profile_name>.cfg.
 e.g. I have two profiles (one for wireless router and one for phone) named tplink.cfg and phone.cfg.

 Example contents of phone.cfg (comments not expected in actual config file):

	wifi_profile=wrouter		# wifi network profile name
	ubuntu_ip=192.168.0.101 	# last assigned ip for Ubuntu machine
	win10_ip=192.168.0.100		# last assigned ip for Windows machine
	dns=192.168.0.1			# DNS server

 The following code snippet first tries to get ip address of Windows machine (using its mac address) using 'ip neigh'.
 The known problem with this approach is that neighbors aren't updated for a while and we don't get the ip address in this step. (Needs to be fixed.)
 So, upon failure, we use the last known stored ip from the config file.
 Of course, we only update the hosts file if it differs from the one already in there. (It saves a sudo password prompt.)


 {% highlight bash %}
 win10ip_cfg=$(grep 'win10' $1.cfg | awk -F= '{print $2}')
 win10ip=$(ip neigh | grep 'f8:bb:aa:cc:dd:fd' | awk '{print $1}')

 if [[ ! $win10ip ]]
 then
 	echo win10 ip not found, setting ip from config
 	win10ip=$win10ip_cfg
 fi
 
 win10ip_hosts=$(grep 'win10' /etc/hosts | awk '{print $1}')
 if [ "$win10ip_hosts" != "$win10ip" ]
 then
 	echo editing hosts file
 	sudo sed -i "s/.*\s\+win10/$win10ip\twin10/" /etc/hosts
 fi
 {% endhighlight %}


### Restart Synergy

 As mentioned above, Synergy server needs to be restarted to pick up the new ip address. So, we kill the running instance, restart it and send 'Alt+F4' to it so that it nicely minimizes to Unity top menu bar.

 {% highlight bash %}
 pkill -x synergy
 pkill -x synergys
 synergy 2>&1 | logger &
 sleep 1
 id=`xdotool search --onlyvisible --name synergy | head -1`
 xdotool windowactivate --sync $id key --clearmodifiers alt+F4
 {% endhighlight %}


### Update Linux IP Address In Windows Hosts File

 As an additional step, if the ip address assigned to Linux machine changes, we edit the Windows hosts file to update it.

 {% highlight bash %}
 ubuntu_ip=$(ifconfig wlan0 | grep -po 'inet addr:.*? ' | awk -f: '{print $2}')
 ubuntu_ip_cfg=$(grep 'ubuntu' $1.cfg | awk -f= '{print $2}')

 if [ "$ubuntu_ip" != "$ubuntu_ip_cfg" ]
 then
 	sed -i -e "s/ubuntu_ip=.*/ubuntu_ip=$ubuntu_ip/" $1.cfg 
 	python win10.py edit_hosts $ubuntu_ip
 fi
 {% endhighlight %}


That's it. I have outlined the major components. You can put them together as needed and write up the Linux shell script that calls them all from one place.

It leaves a lot to be desired.

1. Updating the ip addresses. Both 'ip' and 'arp' are slow to update the neighbors. Using 'dig' to get it from router does not always work either.
2. After switching network for Windows, WinRM session disconnects as Windows is now on a different network. It'll be nice to reconnect and get results.
3. As mentioned already, removing use of sftp to upload files can eliminate dependency on sftp/openssh.


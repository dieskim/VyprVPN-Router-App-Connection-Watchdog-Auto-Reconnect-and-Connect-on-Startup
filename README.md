# VyprVPN-Router-App-Connection-Watchdog-Auto-Reconnect-and-Connect-on-Startup

## 1. Set all the values in the - START - VARIABLES TO SET - section
## 2. Add the following under Administator > Scripts > WAN UP
## 3. Reboot router and see it work

```
#!/bin/sh

#### START - VARIABLES TO SET ####

# set vyprvpn login username
VYPRUSER=your-vyprvpn-account-email@email.com
# set vyprvpn login password
VYPRPASSWORD=your-vyprvpn-account-password
# set router login username
USER=your-router-admin-username
# set router login password
PASSWORD=your-router-admin-password
# set to your router ip address
ROUTER_IP=192.168.100.1
# set connection location - Example: Japan / if location with space add spaces as USA%20-%20Los%20Angeles
VPN_LOCATION=USA%20-%20Los%20Angeles
# set connection protocal Example: OpenVPN-160 / Chameleon
VPN_PROTOCOL=OpenVPN-160
# set how often to check connection in seconds
WATCHDOG_SLEEP_SEC=90
# set LOGIN_STATUS
LOGIN_STATUS=NONE
#### END - VARIABLES TO SET####

# OTHER VARS - DO NOT EDIT BELOW THIS LINE #
while sleep $WATCHDOG_SLEEP_SEC
do
CONNECTION_STATUS=$(wget -O - -q -t 1 http://$USER:$PASSWORD@$ROUTER_IP/user/cgi-bin/vyprvpn.cgi?[%22VYPRVPN_GET_STATUS%22] | sed -e 's/[{}]/''/g' | awk -v RS=',"' -F: '/^status/ {print $2}' | sed 's/\(^"\|"$\)//g')
if [ $CONNECTION_STATUS = "CONNECTED" ]
then
	logger "VPN WATCHDOG - CONNECTED"
else
	if [ $CONNECTION_STATUS = "LOGIN_FAILED" ]
	then
		logger "VPN WATCHDOG - STARTUP LOGIN FAILED"
		while [ $LOGIN_STATUS != "OK" ]
		do
		   logger "VPN WATCHDOG - TRY TO LOGIN"
		   LOGIN_STATUS=$(wget -O - -q -t 1 http://$USER:$PASSWORD@$ROUTER_IP/user/cgi-bin/vyprvpn.cgi?[%22VYPRVPN_LOGIN%22,%22$VYPRUSER%22,%22$VYPRPASSWORD%22] | sed -e 's/[{}"]/''/g' | awk -Fres: '{print $2}' | awk -F, '{print $1}')
		   logger "VPN WATCHDOG - LOGIN_STATUS = $LOGIN_STATUS"
		   sleep 5
		done

	else
		logger "VPN WATCHDOG - DISCONNECTED"
		sleep 5
		CONNECTION_STATUS=$(wget -O - -q -t 1 http://$USER:$PASSWORD@$ROUTER_IP/user/cgi-bin/vyprvpn.cgi?[%22VYPRVPN_GET_STATUS%22] | sed -e 's/[{}]/''/g' | awk -v RS=',"' -F: '/^status/ {print $2}' | sed 's/\(^"\|"$\)//g')
		if [ $CONNECTION_STATUS != "CONNECTED" ]
		then
			logger "VPN WATCHDOG - RECONNECTING"
			eval `wget "http://$USER:$PASSWORD@$ROUTER_IP/user/cgi-bin/vyprvpn.cgi?[%22VYPRVPN_CONNECT%22,%22$VPN_LOCATION%22,%22$VPN_PROTOCOL%22]"`
		fi
	fi
fi
done 2>&1 &
```

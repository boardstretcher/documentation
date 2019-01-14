**Basic session creation (pre-curl commands)**
```
# RDSECRET="<RUNDECK_SECRET>"
# VCENTER="https://my-vcenter.dev.net"
# SESSIONID=$(curl -s -k -X POST -H 'Accept: application/json' --basic -u "myuser@dev.net:$RDSECRET" "$VCENTER/rest/com/vmware/cis/session" | python -c 'import json, sys; print(json.loads(sys.stdin.read())["value"]);')
```

**Base curl command**
```
curl -s -k -X GET -H 'Accept: application/json' -H "vmware-api-session-id: $SESSIONID" "$VCENTER/rest/"
```

**API Documentation**
https://code.vmware.com/apis/62/vcenter-management

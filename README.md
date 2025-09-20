# firmware-updates
Firmware updates pulled from the Spotify OTA server while it was still up.

Manually applying updates is annoying but doable using `swupdate-client`.

# OTA Process
## Step 1
Superbird calls `com.spotify.superbird.ota.check_for_updates` with json:
```
{
   'first_time':True, # True on first boot, false otherwise
   'packages':[
      {
         'name':'superbird-os',
         'version':'8.2.5-release'
      }
   ],
   'serial':'XXXXXXXXXXXX'
}
```
and subscribes to `com.spotify.superbird.ota.package_state`

## Step 2
If there's an update available, app replies to the `check_for_updates` call with the following json:
```
{
   'result':[
      {
         'version':'8.4.4-release', # Whatever the latest fw available is
         'name':'superbird-os',
         'hash':'MD5_of_SWU',
         'url':'URL_to_SWU',
         'critical':False, # Puts the car thing in a special mode where it becomes non-functional until it finishes updating
         'size_bytes':SWU_size,
         'auto_updatable':True
      }
   ]
}
```

## Step 3
When the app is done downloading the update it sends the following as an `EVENT` to `package_state`:
```
{
   'state':'download_success',
   'name':'superbird-os',
   'version':'8.4.4-release', # Whatever the latest fw is
   'hash':'MD5_of_SWU',
   'size':SWU_size
}
```

## Step 4
Superbird starts downloading chunks of an update by calling `com.spotify.superbird.ota.transfer` with the following json:
```
{
   'name':'superbird-os',
   'offset':0,
   'size':31680,
   'version':'8.4.4-release'
}
```
and the app simply replies with `{'data': Binary_of_chunk}`

## Step 5
Superbird silently starts applying the OTA and when it succeeds or fails it calls `com.spotify.superbird.setup.get_state` with json:
```
{
   'current':'finished', # Whether or not first time setup is finished
   'ota':'failed' # 'complete' when OTA is successful
}
```

# Disclaimer
"Spotify", "Car Thing" and the Spotify logo are registered trademarks or trademarks of Spotify AB. Thing Labs is not affiliated with, endorsed by, or sponsored by Spotify AB. All other trademarks, service marks, and trade names are the property of their respective owners.

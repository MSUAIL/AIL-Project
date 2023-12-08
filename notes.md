# Documentation

### Materials

### Procedure

1. Re-image the RPi's Micro SSD with Raspberry OS Full 64 bit
  
  * Used an application called Pi imager to change the OS

2. Installed Z-Wave by following this guide:
  * https://zwave-js.github.io/zwave-js-ui/#/getting-started/other-methods
  * https://zwave-js.github.io/zwave-js-ui/#/ 
  * used some slightly modified commands - only need one of the four .zip files
  ```
    cd ~
    mkdir zwave-js-ui
    cd zwave-js-ui
    # download latest version
    curl -s https://api.github.com/repos/zwave-js/zwave-js-ui/releases/latest  \
    | grep "browser_download_url.*zip" \
    | cut -d : -f 2,3 \
    | tr -d \" \
    | wget -i -
    ### unzip zwave-js-ui-v*.zip
    ### ./zwave-js-ui
  
``````
The zwave instance is locally hosted at: 
* http://localhost:8091  




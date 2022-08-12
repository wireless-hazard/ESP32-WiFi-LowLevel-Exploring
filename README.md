# ESP32 WiFi Low Level Explorin

## Entrée - Looking into the private parts
When developing a WiFi application for the ESP32, there are a common set of necessary steps that need to be followed to get the WiFi interface to work. Those steps to configure your device as a wireless station can be found at [esp-idf's examples folder](https://github.com/espressif/esp-idf/tree/release/v4.4/examples/wifi/getting_started/station), and (at least at the beginning of this journey) that example is going to be the base in which this project is build upon.

The first command to touch the wireless interface configuration is called __esp_wifi_init()__ and receives a configuration struct that MUST be initialized using a macro called **WIFI_INIT_CONFIG_DEFAULT()**. What does that macro do? What are the attributes of this struct? These kind of questions that motivated me to do this deep dive.

Searching into the [esp-idf components folder](https://github.com/espressif/esp-idf/tree/release/v4.4/components/esp_wifi), you're able to find the one related to wifi, called esp_wifi. Navigating through the files I realized that some of the libraries are pre-compiled, and I wouldn't be able to read the whole code base related to the WiFi. But let's see how far can we go!

Luckly, after some search, the file [esp_wifi.h](https://github.com/espressif/esp-idf/blob/release/v4.4/components/esp_wifi/include/esp_wifi.h) has the answer for our first mystery: 


```c
#define WIFI_INIT_CONFIG_DEFAULT() { \
    .event_handler = &esp_event_send_internal, \
    .osi_funcs = &g_wifi_osi_funcs, \
    .wpa_crypto_funcs = g_wifi_default_wpa_crypto_funcs, \
    .static_rx_buf_num = CONFIG_ESP32_WIFI_STATIC_RX_BUFFER_NUM,\
    .dynamic_rx_buf_num = CONFIG_ESP32_WIFI_DYNAMIC_RX_BUFFER_NUM,\
    .tx_buf_type = CONFIG_ESP32_WIFI_TX_BUFFER_TYPE,\
    .static_tx_buf_num = WIFI_STATIC_TX_BUFFER_NUM,\
    .dynamic_tx_buf_num = WIFI_DYNAMIC_TX_BUFFER_NUM,\
    .cache_tx_buf_num = WIFI_CACHE_TX_BUFFER_NUM,\
    .csi_enable = WIFI_CSI_ENABLED,\
    .ampdu_rx_enable = WIFI_AMPDU_RX_ENABLED,\
    .ampdu_tx_enable = WIFI_AMPDU_TX_ENABLED,\
    .amsdu_tx_enable = WIFI_AMSDU_TX_ENABLED,\
    .nvs_enable = WIFI_NVS_ENABLED,\
    .nano_enable = WIFI_NANO_FORMAT_ENABLED,\
    .rx_ba_win = WIFI_DEFAULT_RX_BA_WIN,\
    .wifi_task_core_id = WIFI_TASK_CORE_ID,\
    .beacon_max_len = WIFI_SOFTAP_BEACON_MAX_LEN, \
    .mgmt_sbuf_num = WIFI_MGMT_SBUF_NUM, \
    .feature_caps = g_wifi_feature_caps, \
    .sta_disconnected_pm = WIFI_STA_DISCONNECTED_PM_ENABLED,  \
    .magic = WIFI_INIT_CONFIG_MAGIC\
}
```
A lot of these options are defined in the menuconfig, and the documentation states that you can safely overwrite one of these, but only after you initialize the config struct with that macro.
The file also has the declaration of most (I didn't check if all) of the WiFI related functions provided to the developer.

Ok, I think is time to look for obscured stuff.

Browsing into the esp_private folder, I found the file that actually made me create this repository: [wifi.h](https://github.com/espressif/esp-idf/blob/release/v4.4/components/esp_wifi/include/esp_private/wifi.h).

That file has so many provocative function declarations and, without knowing the future, I would bet that I am going to spend some time trying to use those functions. The wifi.h file is composed by internal API's that vary from custom mallocs, log level changers, callback setters, among other equaly interesting functions.

So, my first test was to include that wifi.h file and use the simplest function possible, only to test if I can actually include it and compile my program.

```c
#include "esp_private/wifi.h"
/*
...
The rest of the example 
...
*/
ESP_ERROR_CHECK(esp_wifi_internal_set_log_level(WIFI_LOG_VERBOSE));
```

The code compiles and runs as expected but, to be honest, nothing seemed to have changed...But it runs!!

At this point I used the function only looking through its interface declared at wifi.h but, where is the wifi.c or equivalent file?

After some guessing, turns out that I think that those [pre-compiled libraries](https://github.com/espressif/esp32-wifi-lib/tree/a224df6c90e2b04c498afaa59bcd538b1e791db9/esp32) mentioned before contain the definition for our functions.

It seemed like a dead end for a while, but a googled a lot and stumbled across a company called [Oryx Embedded](https://www.oryx-embedded.com/), they develop a lot of network related solutions and have a huge list of [open source repositories](https://github.com/Oryx-Embedded).




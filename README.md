# solarspeak

## 1. Dùng để làm gì?

 Biến bất kì chiếc loa cũ nào của bạn thành một loa có kết nối wifi có thể tích hợp vào homeassistant để phát nhạc hoặc thông báo bằng giọng nói

## 2. Kết nối phần cứng

Cần chuẩn bị esp32, module i2s max98357, loa 4 ohm.

Kết nối:

- VCC của max98357 nối với 3V của ESP32
- LRC của max98357 nối với GPIO26 của ESP32
- BCLK của max98357 nối với GPIO27 của ESP32
- DIN của max98357 nối với GPIO25 của ESP32
- GND của max98357 nối với GND của ESP32

## 3. Nạp chương trình

```bash
pip3 install esphome

git clone https://github.com/TThanhXuan/solarspeak

cd solarspeak
# Create a secrets.yaml containing some setup specific secrets
cat > secrets.yaml <<EOF
wifi_ssid: MY_WIFI_SSID
wifi_password: MY_WIFI_PASSWORD
EOF

esphome run solarspeak.yaml

```
## 4 Kết nối với homeassistant

Sao khi nạp chương trình, homeassistant sẽ thông báo tìm thấy device mới, các bạn cứ nhấn thêm theo hướng dẫn

## 5. Tạo thông báo

Bạn có thể tạo bất kì thông báo nào dựa trên vào TTS. Ví dụ:

Cảnh báo mất điện
```yaml
alias: 34_Lux grid off notify
description: ""
triggers:
    - entity_id:
            - sensor.lux_grid_voltage_live
        below: 200
        trigger: numeric_state
actions:
    - service: tts.google_translate_say
        data:
            entity_id: media_player.solarspeak
            message: "Cảnh báo: mất điện lưới"
            language: vi
mode: single
```

Thông báo có điện
```yaml
alias: 35_Lux grid on notify
description: ""
triggers:
  - entity_id:
      - sensor.lux_grid_voltage_live
    above: 200
    trigger: numeric_state
actions:
  - action: tts.speak
    metadata: {}
    data:
      cache: true
      media_player_entity_id: media_player.solarspeak
      message: Điện lưới đã được khôi phục
      language: vi
    target:
      entity_id: tts.google_translate_vi_com
mode: single
```

Cảnh báo tải cao khi mất điện
```yaml
alias: 36_cảnh báo tải cao khi mất điện
description: ""
triggers:
  - trigger: numeric_state
    entity_id:
      - sensor.lux_load_total
    above: 4000
conditions:
  - condition: numeric_state
    entity_id: sensor.lux_grid_voltage_live
    below: 200
actions:
  - action: tts.speak
    metadata: {}
    data:
      language: vi
      media_player_entity_id: media_player.solarspeak
      message: >-
        Cảnh báo: tải cao khi mất điện {{states('sensor.lux_load_total') |
        round(0)}}W, Đề nghị không mở thêm tải
    target:
      entity_id: tts.google_translate_vi_com
mode: single
```




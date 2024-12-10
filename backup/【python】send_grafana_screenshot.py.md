- config.py
```python
# Grafana配置
GRAFANA_URL = ""
GRAFANA_API_KEY = ""
DASHBOARD_UID = ""
DASHBOARD_NAME = ""
PANEL_ID = 12
ORG_ID = 1

# 企业微信配置
WECOM_WEBHOOK = ""

# 截图配置
SCREENSHOT_WIDTH = 1000
SCREENSHOT_HEIGHT = 500
TIME_RANGE = "1h"
TIMEZONE = "Asia/Shanghai"
THEME = "light" 
```
- send_grafana_screenshot.py
```python
import requests
import time
import os
import base64
import hashlib
import config

def get_grafana_screenshot():
    headers = {
        'Authorization': f'Bearer {config.GRAFANA_API_KEY}'
    }
    
    render_url = f"{config.GRAFANA_URL}/render/d-solo/{config.DASHBOARD_UID}/{config.DASHBOARD_NAME}"
    params = {
        'orgId': config.ORG_ID,
        'panelId': config.PANEL_ID,
        'width': config.SCREENSHOT_WIDTH,
        'height': config.SCREENSHOT_HEIGHT,
        'tz': config.TIMEZONE,
        'from': f'now-{config.TIME_RANGE}',
        'to': 'now',
        'theme': config.THEME
    }
    
    response = requests.get(render_url, headers=headers, params=params)
    if response.status_code != 200:
        return None
        
    filename = f"grafana_screenshot_{int(time.time())}.png"
    with open(filename, 'wb') as f:
        f.write(response.content)
    return filename

def send_to_wecom(screenshot_path):
    with open(screenshot_path, 'rb') as f:
        image_data = f.read()
        base64_data = base64.b64encode(image_data).decode('utf-8')
        md5_data = hashlib.md5(image_data).hexdigest()
    
    data = {
        "msgtype": "image",
        "image": {
            "base64": base64_data,
            "md5": md5_data
        }
    }
    
    response = requests.post(config.WECOM_WEBHOOK, json=data)
    response_json = response.json()
    if 'errcode' in response_json and response_json['errcode'] != 0:
        return False
    return True

def main():
    screenshot = get_grafana_screenshot()
    if screenshot:
        if send_to_wecom(screenshot):
            print("截图发送成功！")
        os.remove(screenshot)

if __name__ == "__main__":
    main() 
```
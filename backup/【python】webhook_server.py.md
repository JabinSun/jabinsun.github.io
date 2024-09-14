```python
import json
import os
import subprocess
import uuid

from flask import Flask, jsonify, request

app = Flask(__name__)

WEBHOOK_KEYS_FILE = '.webhook_keys.json'


def load_webhook_keys():
    if os.path.exists(WEBHOOK_KEYS_FILE):
        with open(WEBHOOK_KEYS_FILE, 'r') as f:
            return json.load(f)
    return {}


def save_webhook_keys(keys):
    with open(WEBHOOK_KEYS_FILE, 'w') as f:
        json.dump(keys, f)


webhook_keys = load_webhook_keys()


@app.route('/getWebhookKey', methods=['GET'])
def get_webhook_key():
    key = str(uuid.uuid4())
    webhook_keys[key] = True
    save_webhook_keys(webhook_keys)
    return jsonify({'webhook_key': key}), 200


@app.route('/webhook/<key>', methods=['POST'])
def handle_webhook(key):
    if key in webhook_keys and webhook_keys[key] is True:
        try:
            data = request.get_json()
            args = data.get('shell', [])
            if not isinstance(args, list):
                return jsonify({'status': 'error', 'error': 'Arguments should be a list'}), 400
            result = subprocess.run(args,
                                    check=True,
                                    stdout=subprocess.PIPE,
                                    stderr=subprocess.PIPE)
            return jsonify({'status': 'success', 'output': result.stdout.decode('utf-8')}), 200
        except subprocess.CalledProcessError as e:
            return jsonify({'status': 'error', 'error': e.stderr.decode('utf-8')}), 500
    else:
        return jsonify({'status': 'error', 'error': 'Invalid webhook key'}), 403


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
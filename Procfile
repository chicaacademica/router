from flask import Flask, request, Response, jsonify
import requests
import os
from urllib.parse import urljoin
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# Get BASE_URL from environment variable, fallback to something safe
BASE_URL = os.getenv('BASE_URL')

@app.route('/', defaults={'path': ''}, methods=['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'])
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'])
def proxy(path):
    # Full URL to forward to
    target_url = urljoin(BASE_URL.rstrip('/') + '/', path)

    # Forward query params
    if request.query_string:
        target_url += '?' + request.query_string.decode()

    # Prepare headers (excluding 'host')
    headers = {key: value for key, value in request.headers if key.lower() != 'host'}

    try:
        # Proxy the request
        resp = requests.request(
            method=request.method,
            url=target_url,
            headers=headers,
            data=request.get_data(),
            cookies=request.cookies,
            allow_redirects=False,
            stream=True
        )

        # Exclude certain headers
        excluded_headers = {'content-encoding', 'content-length', 'transfer-encoding', 'connection'}
        response_headers = [(name, value) for name, value in resp.raw.headers.items() if name.lower() not in excluded_headers]

        return Response(resp.content, status=resp.status_code, headers=response_headers)

    except requests.exceptions.RequestException as e:
        return jsonify({"error": str(e)}), 500


if __name__ == '__main__':
    app.run(debug=True)

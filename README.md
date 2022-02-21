# grab-logs

## How to use it
```
python3 grab_log.py --path /var/log/redis --delimiter "#,*" --folder --format json --output /home/osu
python3 grab_log.py --path /var/log/journal --delimiter "#,*" --folder --output /home/osu
python3 grab_log.py --path /var/log/mongodb --delimiter "I" --folder --output /home/osu --format json
python3 grab_log.py --path /var/log/docker.log --delimiter "level=" --single-file --output /home/osu
python3 grab_log.py --path /var/log/dpkg.log --delimiter "\n" --single-file --format json --output /home/osu
```

## Code
```python
from pathlib import *
from json import dump
from tqdm import tqdm
import argparse
import os
import sys


formatfile = ['txt','text','log','data']


def export_to_json(data):
    arr_object = []
    for i in range(len(data)):
        arr_object.append({
            "timestamp":data[i][0],
            "message":data[i][1],
        })
    with open(output_filename, 'w', encoding='utf-8') as f:
        dump(arr_object, f, ensure_ascii=False, indent=4)
        f.close()


def export_to_log(data):
    with open(output_filename, 'w', encoding='utf-8') as f:
        for line in data:
            f.writelines(
                line[0].strip()+';'+line[1].strip()+'\n'
            )
        f.close()


def finder(filename, format):
    timestamps = []
    messages = []

    if opt.folder:
        path = ''.join((filename,'/')) if '/' not in filename[-1] else filename
        files = os.listdir(path)

        if len(files) == 0:
            print('There is no files ...')
            sys.exit()
        
        for data in files:
            with open(path+data,'r') as f:
                lines = f.readlines()
                for split in delimiter:
                    for line in tqdm(lines, desc = 'Splitting ... %s' %split):
                        log = line.strip().split(split)

                        timestamps.append(log[0])
                        messages.append(''.join(log[1:]))
    else:       
        with open(filename,'r') as f:
            lines = f.readlines()
            for split in delimiter:
                for line in tqdm(lines, desc = 'Splitting ... %s' %split):
                    log = line.strip().split(split)

                    timestamps.append(log[0])
                    messages.append(''.join(log[1:]))

    f.close()
    data = list(zip(timestamps, messages))

    if format in formatfile:
        export_to_log(data)

    elif format == 'json':
        export_to_json(data)

    else:
        print('Format is not supported')
        sys.exit()



def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        '--path', '-p', type=str, help='Path to log folder',default=argparse.SUPPRESS
    )
    parser.add_argument(
        '--output', '-o', type=str, help='Path to output folder',default=argparse.SUPPRESS
    )
    parser.add_argument(
        '--format', '-f', type=str, help='Format output file %s'%formatfile, default='txt'
    )
    parser.add_argument(
        '--delimiter', '-d', type=str, help="Separate data by [',',';','#','/','\\n','\\t']. Multiple delimiter \"#,*\"", default='#'
    )
    parser.add_argument('--folder', dest='folder', action='store_true', help='Grab all logs in folder')
    parser.add_argument('--single-file', dest='folder', action='store_false', help='Grab single file')
    parser.set_defaults(folder=True)
    opt = parser.parse_args()
    return opt


opt = main()

delimiter = [str(d) for d in opt.delimiter.split(',')]
format = opt.format.replace('.','') if '.' in opt.format else opt.format
output_filename = ''.join((opt.output,'/')) if '/' not in opt.output[-1] else opt.output
output_filename = output_filename+'output.'+format

finder(opt.path, format)
print(output_filename, " Done ...")
```

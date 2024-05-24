# Session 60 - Project 6 - Data Analytics with Hadoop Project

## Tools
- Postgres (via Docker or direct install)
- Hadoop versi < 3.2.1
- Script Project 3 - Batch Processing
- GUI Postgres (Pgadmin, Dbeaver, dll) (optional)

## Preparation
1. Download script proyek terkait batch processing sebelumnya, link download
2. Pahami script project diatas terlebih dahulu (Tutor akan membantu menjelaskan)
3. Install hadoop (Harusnya sudah di Install pada sesi Hadoop)
    - tutorial cara menginstall hadoop untuk ubuntu https://www.rosehosting.com/blog/how-to-install-hadoop-on-debian-11/
    - Installing Hadoop 3.2.1 Single Node Cluster on Windows 10 untuk windows https://towardsdatascience.com/installing-hadoop-3-2-1-single-node-cluster-on-windows-10-ac258dd48aef
    - Hadoop Single Node Cluster on Docker via docker https://medium.com/analytics-vidhya/hadoop-single-node-cluster-on-docker-e88c3d09a256
    - can run with this command:
    ```
    docker run -d -it --name hadoop-local -p 9864:9864 -p 9870:9870 -p 8088:8088 --hostname hadoop-local hadoop
    ```
4. Jangan lupa untuk tambahkan host nya
    ```
    127.0.0.1 hadoop-local
    35.222.31.142 hadoop-server
    ```
5. Add hosts in device windows:
    - localhost custom domain name https://ecompile.io/blog/localhost-custom-domain-name
6. Setelah install Hadoop, Pastikan apps dibawah ini running:
    - Hadoop Namenode
    - Hadoop datanode
    - YARN Resource Manager
    - YARN Node Manager
7. 3 Optional untuk menggunakan hadoop dalam project ini:
    - install hadoop di os device masing2
    - install hadoop via docker
    - menggunakan hadoop server dari tutor http://35.222.31.142:9870 (temporary)
8. (Optional) Sambungkan koneksi database postgres dengan GUI yang kalian punya di device masing - masing

## Flow Project 4 - Data Analytics with Hadoop:
Link to Flow Diagram https://drive.google.com/file/d/1rzhm4n4sy4_668xCfLXOlJKhUkj0HV2j/view?usp=sharing

## Study Case
Di proyek sebelumnya anda telah membuat batch processing untuk data migrasi. Sebagai tambahan, anda diminta untuk membuat tabel data mart yang nantinya akan digunakan oleh data analis untuk membuat dashboard report order setiap bulannya. Proses yang dilakukan tidak boleh lebih dari 1 jam.

Requirement tabel data mart:
- output columns: month, total_orders

Setelah memahami tugas yang diberikan, kita bisa menjabarkan task - task apa saja yang harus dikerjakan, task tersebut sebagai berikut:
1. Extract data source ke database postgres (skip, sudah ada di project 3).
2. Buatlah file DWH dim_orders dengan menggunakan hadoop.
3. Buatlah data mart total_orders_based_on_month dengan MapReduce, data diambil dari DWH.

## Step by Step
Proyek ini melanjutkan proyek sebelumnya terkait batch processing. (Silakan dibuka script yang telah disiapkan sebelumnya)
1. Buatlah flow diagram ETL yang akan dibuat
2. Tambahkan koneksi json hadoop
3. Buatlah script koneksi hadoop
4. Define datetime now
5. Buatlah script data dummy DWH ke local terlebih dahulu
6. Buatlah script upload data ke hadoop sebagai DWH
7. Buatlah script transform dengan menggunakan MapReduce
8. Sesuaikan output yang diminta

## MapReduce.py
```python
from mrjob.job import MRJob
from mrjob.step import MRStep

import csv
import json

cols = 'order_id,order_date,user_id,payment_name,shipper_name,order_price,order_discount,voucher_name,voucher_price,order_total,rating_status'.split(',')

def csv_readline(line):
    for row in csv.reader([line]):
        return row

class OrderDateCount(MRJob):
    def steps(self):
        return [ 
            MRStep(mapper=self.mapper, reducer=self.reducer),
            MRStep(reducer=self.sort)
        ]

    def mapper(self, _, line):
        row = dict(zip(cols, csv_readline(line)))

        if row['order_id'] != 'order_id':
            yield row['order_date'][0:7], 1

    def reducer(self, key, values):
        yield None, (key, sum(values))

    def sort(self, key, values):
        data = []
        for order_date, order_count in values:
            data.append((order_date, order_count))
            data.sort()
        
        for order_date, order_count in data:
            yield order_date, order_count

if __name__ == '__main__':
    OrderDateCount.run()
```

## Config.json
```json
{
    "marketplace_prod": {
        "host": "35.222.31.142",
        "db": "data_source",
        "user": "postgres",
        "password": "sib6_admin",
        "port": "5436"
    },
    "dwh": {
        "host": "35.222.31.142",
        "db": "dwh",
        "user": "postgres",
        "password": "sib6_admin",
        "port": "5436"
    },
    "hadoop":{
        "client":"http://hadoop-server:9870"
    }
}


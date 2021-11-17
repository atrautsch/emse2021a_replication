# Replication Kit

We provide two entrypoints for this replication kit.
The first one starts with the SmartSHARK MongoDB dump and extracts PMD Rules and just-in-time defect prediction data. The second one just uses the collected data to re-create the plots and tables used in the paper.


## Create Data
In this entrypoint, we load the MongoDB dump and use it to extract just-in-time defect prediction data.

### Load MongoDB Dump

The MongoDB database dump we are using is [SmartSHARK 2.1](https://smartshark.github.io/dbreleases/) as it contains the data from our tangling study.

```bash
wget https://doi.org/10.25625/7OZ1SP
mongorestore -uUSER -p'PASSWORD' --gzip --archive=smartshark_2_1.agz
```

### Extract Just-in-time defect prediction data

We use [Gierlappen](https://github.com/atrautsch/Gierlappen) which utilizes the SmartSHARK database, the stored repository data as well as PMD to create the data.

```bash
# install Gierlappen
git clone https://github.com/atrautsch/Gierlappen.git
cd Gierlappen
python3 -m venv .
source bin/activate
pip install -r requirements.txt

# install PMD for extraction of warnings
wget https://github.com/pmd/pmd/releases/download/pmd_releases%2F6.31.0/pmd-bin-6.31.0.zip
unzip pmd-bin-6.31.0.zip -d ./checks/
mv checks/pmd-bin-6.31.0/* ./checks/pmd/

# extract repository data from smartshark, has to be run for every project
python smartshark_dump_repository.py --project ant-ivy --path /srv/repos/ --db-host 127.0.0.1 --db-port 27017 --db-name smartshark_2_1 --db-user USER --db-pw PW --db-auth smartshark_2_1

# extract just-in-time defect prediction data from the database and repository, has to be run for every project
python smartshark_mining.py --project ant-ivy --path /srv/repos/ant-ivy/ --label-name JLMIVLV --db-host 127.0.0.1 --db-port 27017 --db-name smartshark_2_1 --db-user USER --db-pw PW --db-auth smartshark_2_1
```


## Re-create the Plots and Tables
This uses all collected data to create the plots and tables.
We use jupyter notebooks for this task.

### Download just-in-time defect prediction data

```bash
wget https://mediocre.hosting/jit_data.tar.gz
tar -xzf jit_data.tar.gz
```

### Create virtualenv and install dependencies
```bash
python3 -m venv .
pip install -r requirements.txt
cd notebooks
jupyter lab
```

The plots and tables can then be re-recreated by running all cells in the PlotsTables notebook.

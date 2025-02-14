# Install Hypopg for PostgreSQL 16 - Mac M1

Download source code zip from https://github.com/HypoPG/hypopg/releases

Uncompress the zip file and `cd` into it. Then `sudo make install`.

```bash
cd ~/Downloads/hypopg-1.4.1
sudo make install
```


## Enable extension

Go to `psql` shell and enable the extension:
```bash
CREATE EXTENSION hypopg;
```
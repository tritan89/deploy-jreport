# JReport 15.6 dq203 deployment

Replicates the dq201 JReport server (CLD1LDAPP222) as a standalone instance pointed at dq203 databases.

## 1. Stage tarballs on the source (dq201)

```
sudo systemctl stop jreport
sudo tar --exclude='Server/logs' --exclude='Server/temp' --exclude='Server/history' -czf /tmp/jreport-server.tar.gz -C /opt JReport
sudo tar -czf /tmp/jdk1.8.0.tar.gz -C /opt jdk1.8.0
sudo tar -czf /tmp/jreport-artifacts.tar.gz -C /opt jreport_artifacts
sudo systemctl start jreport
```

Then copy all three into `files/` on the control machine (run from the repo root):

```
mkdir -p files
scp tlanger@10.123.0.141:/tmp/jreport-server.tar.gz files/
scp tlanger@10.123.0.141:/tmp/jdk1.8.0.tar.gz files/
scp tlanger@10.123.0.141:/tmp/jreport-artifacts.tar.gz files/
```

Once copied, `files/` should contain `jreport.service` plus the three tarballs. Then optionally remove the staged copies on dq201 (`sudo rm /tmp/jreport-server.tar.gz /tmp/jdk1.8.0.tar.gz /tmp/jreport-artifacts.tar.gz`).

## 2. Secrets

```
cp vault.yml.example vault.yml
ansible-vault encrypt vault.yml
```

If dq203 MySQL credentials are identical to dq201, copy the four encrypt-sign values from the source `dbconfig.xml` and `datasource.xml`. If they differ, deploy with placeholders, then set real credentials post-deploy via the JReport admin console or DBMaintain.sh before starting the service.

## 3. Prerequisites on dq203 side

- MySQL schemas exist on the dq203 DB host: `jreportsystem`, `jreportrealm`, `jreportprofiling` (empty for a fresh instance, or restored from dq201 dumps to carry over users/schedules/folders)
- Network path open from the new app server to dq203 MySQL on 3306
- Host has at least 24GB RAM (JVM heap is fixed at 20480MB)

## 4. Run

Dry run and deploy without starting:

```
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass --check
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

Verify config on the target, then start:

```
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass -e jreport_start_service=true
```

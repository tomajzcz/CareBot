# CareBot
Junior DevOps Engineer Technical Assessment

////////////////////////////////////////////////////
Ansible je vytvořený pro konfiguraci debian serveru.
////////////////////////////////////////////////////

Máme připravený čistý server, známe IP adresu, kterou si zapíšeme do hosts souboru. Jako první spustíme playbook DockerEnvironment.yml, který nakonfiguruje server, zabezpečí, vytvoří uživatele carebot a připraví server ke spuštění dockeru.

V roles/DockerEnvironment se nachází 3 adresáře pro konfiguraci serveru. V adresáři files jsou soubory potřebné ke konfiguraci, v adresáři handlers se nachází playbook pro restart a povolení služeb. Ve třetím adresáři tasks se nachází samotné konfigurační playbooky, kdy se spustí hlavní main.yml, který následně určuje pořadí spouštění dalších pomocí import_tasks.

packages.yml 
- updatne se balíkovací systém a nainstalují se vypsané balíčky

docker.yml
- nainstaluje se pomocí přidaného repository docker
- v bloku "Configure Docker logging" se nakonfiguruje daemon.json pro maximální velikost a rotaci logů

user.yml
- vytvoření uživatele - heslo, authorized_keys soubor pro ssh klíče
- možnost přidat do sudo group - zde zakomentovaná

sudoers.yml
- aby mohl uživatel carebot pracovat s dockerem a zároveň nebyl v sudo group, je nutné tyto příkazy pro daného uživatele povolit v /etc/sudoers
- vyřešil jsem to přidáním konfiguračního souboru do /etc/sudoers.d/docker, povolil jsem příkazy docker ps, start, stop, logs

sshd.yml
- konfigurace sshd pro zabezpečení ssh
- je možné službu restartovat, nicméně tím si zamezím root přístup na server a tím další konfiguraci - nechal jsem restart sshd zakomentovaný pro potřeby testování
	- na produkci by se buď musel upravit sshd_config pro přístup roota z konfiguračního serveru nebo vytvořit nějakého ansible konfiguračního uživatele s přístupem na roota

firewall.yml
- konfigurace firewallu iptables, kdy je defaultně povolený jen ping a ssh port 671

Tímto je základní konfigurace serveru hotova.

Playbooky OrthancCompose.yml a OrthancDeployment.yml slouží ke stejné věci - nasazení Orthanc aplikace a spuštění přes docker. V prvním případě se jedná o nasazení přes docker-compose.yml, v tom druhém je čistě jen spuštění daného kontejneru na serveru, bez souboru docker-compose.yml.

Způsob konfigurace ansible je stejný jako s DockerEnvironment playbookem. Tj. main.yml, který importuje zbylé tasky z roles/tasks/

Oba playbooky na nasazení aplikace začínají stejně - konfigurací firewallu a povolení portů 4242 a 8042. Následně se importuje task orthanc.yml, kde dojde k nasazení aplikace. Vytvoření adresáře /opt/orthanc, nakopírování json konfiguračních souborů, spuštění docker kontejneru nebo nakopírování a spuštění docker-compose.yml a ověření, zda aplikace přes web běží a je vše v pořádku.

////////////////////////////////////////////////////

Největší výzva pro mě byla vytvoření docker kontejneru pomocí ansiblu. S tímto jsem zatím praktickou zkušenost neměl, nicméně jsem se s tím popral. Vytvořil jsem nejdříve ansible, který jenom spustí docker kontejner. Jakmile vše fungovalo, napsal jsem druhý ansible, který vytvoří docker-compose.yml v /opt/orthanc/, aby se v případě potřeby mohl konfigurovat přehledněji nebo ručně na serverech.

Ansible jsem testoval na Hetzner cloud serveru - na svém PC k tomu nemám úplně dobré podmínky, co se týče výkonu. Ansible jsem psal ve VSCode, následně nahrál na git.

Pokud bych potřeboval deploynout druhý, identický server, mám několik možností... První možností je přidání serveru do ansible inventory a konfigurace ansiblem, tak jako server první. Druhá možnost - v případě, že by se jednalo o virtuální server je možnost server naklonovat, upravit hostname, IP a znovu spustit. Nebo vytvořit z virtuálního serveru template a tu v případě potřeby nasazovat.

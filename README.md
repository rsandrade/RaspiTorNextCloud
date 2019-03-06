# Raspiberry + Tor + NextCloud
Instalação de NextCloud com dados criptografados e acessíveis via Tor

Esse tutorial explica o caminho para criação de um repositório de documentos acessível pela rede Tor.

## Materiais utilizados:

- Raspberry Pi 3 B+ (outros Raspberries devem funcionar)
- HD Externo (500 MB ou maior recomendado)

## Passos a serem realizados:

1) Instalar o Raspbian Stretch

Siga: https://www.raspberrypi.org/downloads/raspbian/

2) Instalar o HD Externo

Com o Raspbian instalado, assumindo que apenas o HD Externo está conectado ao Raspberry e que este está em `/dev/sda`, siga os comandos:

```
sudo fdisk /dev/sda 
# Press 'd'
# Press 'w'
```
Create new primary partition.

```
sudo fdisk /dev/sda
# Press 'n'
# Press 'p'
# Press '1'
# Press 'enter'
# Press 'enter'
# Press 'w'
```
Format partition as ext4.
```
sudo mkfs.ext4 /dev/sda1
```

Create folder for mount.
```
sudo mkdir /mnt/hd
```

Look up UUID of flash drive.
```
UUID="$(sudo blkid -s UUID -o value /dev/sda1)"
```

Add mount to fstab.
```
echo "UUID=$UUID /mnt/hd ext4 defaults,nofail 0" | sudo tee -a /etc/fstab
```

Test fstab file.
```
sudo mount -a
```

Check to see if drive is mounted.
```
df -h
```
/dev/sda1 should appear as mounted on /mnt/hd

3) Instale o Snapd

```
sudo apt install snapd
```

4) Instale o Nextcloud via Snapd

```
sudo snap install nextcloud
```

5) Habilitando o Hidden Services do Tor

Siga: https://mroystonward.github.io/self-hosting-with-raspberry-pi-and-tor/#installing-tor

Na configuração do Hidden Service, use `HiddenServicePort 80 127.0.0.1:80`, ou seja aponte para porta 80 e não para a 7658, como no tutorial do link.

Reinicie o Tor (`sudo service restart tor`) e verifique seu endereço .onion, com o qual acessará seu Nextcloud, com `sudo cat /var/lib/tor/hidden_service/hostname`. Guarde esse endereço para que ele seja adicionado aos "trusted domains".

6) Adicionando o .onion aos "trusted domains"

Acesse as configurações do Nextcloud com `sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php` e na parte do trusted_domains adicione o endereço .onion, de modo a ficar semelhante ao exemplo abaixo:

```
'trusted_domains' => 
  array (
    0 => '192.168.1.3',
    1 => 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion',
  ),
```

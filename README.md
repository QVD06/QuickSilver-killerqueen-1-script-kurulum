# QuickSilver-killerqueen-1-script-kurulum


<p style="font-size:14px" align="right">
<a href="https://t.me/berkcak" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/50621007/168689534-796f181e-3e4c-43a5-8183-9888fc92cfa7.png" width="30"/></a>
<a href="https://discord.gg/t5WUEJbh" target="_blank">Visit our website <img src="https://user-images.githubusercontent.com/50621007/168689709-7e537ca6-b6b8-4adc-9bd0-186ea4ea4aed.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/166148846-93575afe-e3ce-4ca5-a3f7-a21e8a8609cb.png">
</p>

# Quicksilver testnet düğüm kurulumu — killerqueen-1

Resmi belgeler:

>- [Doğrulayıcı Kurulum Talimatları](https://github.com/ingenuity-build/testnets)

Gezgin:
>-  https://quicksilver.explorers.guru/


## Donanım Gereksinimleri
Herhangi bir Cosmos-SDK zinciri gibi, donanım gereksinimleri de oldukça mütevazı.

### Minimum Donanım Gereksinimleri
 - 3x CPUs
 - 4GB RAM
 - 80GB Disk

### Önerilen Donanım Gereksinimleri
 - 4x CPUs
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 
## Quicksilver fullnode'unuzu kurun

### Otomatik Kurulum
Aşağıdaki otomatik komut dosyasını kullanarak Quicksilver fullnode'unuzu birkaç dakika içinde kurabilirsiniz. Doğrulayıcı düğüm adınızı girmenizi isteyecektir!
```
wget https://raw.githubusercontent.com/berkcaNode/QuickSilver-killerqueen-1-script-kurulum/main/killerqueen-1.sh && bash killerqueen-1.sh
```


## Yükleme sonrası Adımları

Kurulum bittiğinde lütfen değişkenleri sisteme yükleyin

```
source $HOME/.bash_profile
```

Ardından, doğrulayıcınızın blokları senkronize ettiğinden emin olmalısınız. Senkronizasyon durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
quicksilverd status 2>&1 | jq .SyncInfo
```

### Cüzdan oluştur
Yeni cüzdan oluşturup katılamayacağınız için yeni cüzdan oluşturmayı paylaşmıyorum. Gentx formunu doldururken vermiş olduğunuz cüzdanınızı mnemoniclerinizi kullanarak recover etmelisiniz..

Cüzdanınızı menmoniclerinizi kullanarak kurtarmak için
```
quicksilverd keys add $WALLET --recover
```

Mevcut cüzdan listesini almak için

```
quicksilverd keys list
```

### Cüzdan bilgilerini kaydet

Cüzdan ve valoper adresi ekleyin, değişkenleri sisteme yükleyin

```
QUICKSILVER_WALLET_ADDRESS=$(quicksilverd keys show $WALLET -a)
QUICKSILVER_VALOPER_ADDRESS=$(quicksilverd keys show $WALLET --bech val -a)
echo 'export QUICKSILVER_WALLET_ADDRESS='${QUICKSILVER_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export QUICKSILVER_VALOPER_ADDRESS='${QUICKSILVER_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Cüzdanınıza jeton tahsis edin

Doğrulayıcı oluşturmak için önce cüzdanınıza testnet jetonları ile para yatırmanız gerekir. Cüzdanınızı doldurmak için QCK ve ATOM musluklarına erişmek için Quicksilver discord sunucusuna katılın. Uygun kanalda olduğunuzdan emin olun
- **#qck-tap** QCK jetonları için
- **#atom-tap** ATOM jetonları için

Musluk adresini kontrol etmek için:

```
$<YOUR_WALLET_ADDRESS> rhapsody
```

Bakiyenizi kontrol etmek için:

```
$balance <YOUR_WALLET_ADDRESS> rhapsody
```

Musluktan jeton talep etmek için:

```
$request <YOUR_WALLET_ADDRESS> rhapsody
```

### Doğrulayıcı oluştur

Doğrulayıcı oluşturmadan önce lütfen en az 1 qck'e sahip olduğunuzdan (1 qck 1000000 uqck'e eşittir) ve düğümünüzün senkronize edildiğinden emin olun

Cüzdan bakiyenizi kontrol etmek için:

```
quicksilverd query bank balances $QUICKSILVER_WALLET_ADDRESS
```
> Cüzdanınız muhtemelen daha fazla bakiye göstermiyorsa, düğümünüz hala eşitleniyordur. Lütfen senkronizasyonun bitmesini bekleyin ve ardından devam edin 

Aşağıdaki doğrulayıcı çalıştırma komutunuzu oluşturmak için

```
quicksilverd tx staking create-validator \
  --amount 1000000uqck \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(quicksilverd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $QUICKSILVER_CHAIN_ID
```


### Doğrulayıcıların listesini alın

```
quicksilverd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Şu anda bağlı olan eşler listesini kimlikleri ile alın

```
curl -sS http://localhost:${QUICKSILVER_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Faydalı komutlar

### Servis Yönetimi

Günlükleri kontrol et
```
journalctl -fu quicksilverd -o cat
```

Servisi başlat
```
sudo systemctl start quicksilverd
```

Servisi durdur
```
sudo systemctl stop quicksilverd
```

Servisi yeniden başlat
```
sudo systemctl restart quicksilverd
```

### Düğüm bilgisi

Senkronizasyon bilgisi

```
quicksilverd status 2>&1 | jq .SyncInfo
```

Doğrulayıcı bilgisi

```
quicksilverd status 2>&1 | jq .ValidatorInfo
```

Düğüm bilgisi

```
quicksilverd status 2>&1 | jq .NodeInfo
```

Düğüm kimliğini göster

```
quicksilverd tendermint show-node-id
```

### Cüzdan işlemleri
cüzdan listesi

```
quicksilverd keys list
```

Cüzdanı kurtar

```
quicksilverd keys add $WALLET --recover
```

Cüzdanı sil

```
quicksilverd keys delete $WALLET
```

Cüzdan bakiyesini görmek için
```
quicksilverd query bank balances $QUICKSILVER_WALLET_ADDRESS
```

Para transferi
```
quicksilverd tx bank send $QUICKSILVER_WALLET_ADDRESS <TO_QUICKSILVER_WALLET_ADDRESS> 10000000uqck
```

### oylama
```
quicksilverd tx gov vote 1 yes --from $WALLET --chain-id=$QUICKSILVER_CHAIN_ID
```

###Stake, Delegasyon ve Ödüller
Delegate stake
```
quicksilverd tx staking delegate $QUICKSILVER_VALOPER_ADDRESS 10000000uqck --from=$WALLET --chain-id=$QUICKSILVER_CHAIN_ID --gas=auto
```

Payını doğrulayıcıdan başka bir doğrulayıcıya yeniden devretme

```
quicksilverd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uqck --from=$WALLET --chain-id=$QUICKSILVER_CHAIN_ID --gas=auto
```

Tüm ödülleri geri çek

```
quicksilverd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$QUICKSILVER_CHAIN_ID --gas=auto
```

Komisyon ile ödülleri geri çekin

```
quicksilverd tx distribution withdraw-rewards $QUICKSILVER_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$QUICKSILVER_CHAIN_ID
```

### Doğrulayıcı yönetimi

Doğrulayıcıyı düzenle

```
quicksilverd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=1C5ACD2EEF363C3A \
  --website="http://kjnodes.com" \
  --details="Providing professional staking services with high performance and availability. Find me at Discord: kjnodes#8455 and Telegram: @kjnodes" \
  --chain-id=$QUICKSILVER_CHAIN_ID \
  --from=$WALLET
```

Hapisten çıkma 
```
quicksilverd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$QUICKSILVER_CHAIN_ID \
  --gas=auto
```

### Düğümü sil
Bu komutlar düğümü sunucudan tamamen kaldıracaktır. Kendi sorumluluğunuzda kullanın!
```
sudo systemctl stop quicksilverd
sudo systemctl disable quicksilverd
sudo rm /etc/systemd/system/quicksilver* -rf
sudo rm $(which quicksilverd) -rf
sudo rm $HOME/.quicksilver* -rf
sudo rm $HOME/quicksilver -rf
```

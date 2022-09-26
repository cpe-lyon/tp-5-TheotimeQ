**Quere Theotime 3ICS**

# Table des matières

[Exercice 1. Disques et partitions](#Anch1)

[Exercice 2. Partitionnement LVM](#Anch2)

# Exercice 1. Disques et partitions  <a id='Anch1'></a>

## 1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM

![](/TP5/IMG_1.png)

## 2. Vérifiez que ce nouveau disque dur est bien détecté par le système

```console
User@localhost:~$ lsblk
```

![](/TP5/IMG_2.png)

## 3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),et une seconde partition de 3 Go en NTFS (n°7)

```console
User@localhost:~$ sudo fdisk /dev/sdb
```

![](/TP5/IMG_3.png)

## 4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)

```console
User@localhost:~$ sudo mkfs /dev/sdb1
User@localhost:~$ sudo mkfs.ntfs /dev/sdb2
```

![](/TP5/IMG_4.png)

## 5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

```console
User@localhost:~$ df -T
```
La commande df -T ne fonctionne pas sur notre disque car les partitions ne sont encore pas montées

## 6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

On ajoute les deux lignes suivantes au fichier `/etc/fstab`

`/dev/sdb1 /data ext4 defaults 0 0`
`/dev/sdb2 /win ntfs defaults 0 0`

![](/TP5/IMG_5.png)

## 7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration

```console
User@localhost:~$ sudo mount /dev/sdb1  /data
User@localhost:~$ sudo mount /dev/sdb2  /win
User@localhost:~$ sudo mount -a
```

## 8. Montez votre clé USB dans la VM

Je suis sur VSPHERE , impossible de brancher une clef.

Mais la commande devrait être la suivante
```console
User@localhost:~$ sudo mount /dev/usb  /data1
```

## 9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox).

Impossible sur VSPHERE .

# Exercice 2. Partitionnement LVM  <a id='Anch2'></a>

Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler
les disques et les partitions.

## 1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab

On demonte les disques , puis on edite le fichier fstab et on supprime les lignes ajoutées.
```console
User@localhost:~$ sudo umount /data
User@localhost:~$ sudo umount /win
User@localhost:~$ sudo nano /etc/fstab
```

## 2. Supprimez les deux partitions du disque, et créez une partition unique de type LVM


On supprime les volumes fait precedemments 

```console
User@localhost:~$ sudo fdisk /dev/sdb
```

![](/TP5/IMG_6.png)

Pour creer le LVM ( Logical Volume Manager ) , on lui donne le type "8"

![](/TP5/IMG_7.png)

## 3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.

On fait un `man pvcreate` pour regarder comment ca fonctionne.
Cette commande sert a initialiser des volume physique en LVM
On fait donc : 

```console
User@localhost:~$ sudo pvcreate PV /dev/sdb1
User@localhost:~$ sudo pvdisplay
```

![](/TP5/IMG_8.png)

## 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay

D'apres le man , la commande `vgcreate` creer des groupes de volumes

```console
User@localhost:~$ sudo vgcreate VG_new /dev/sdb1
```

![](/TP5/IMG_9.png)

## 5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.



## 6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.

## 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.

## 8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes

## 9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.


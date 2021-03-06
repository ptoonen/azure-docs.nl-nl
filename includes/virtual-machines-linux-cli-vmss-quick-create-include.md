Als u dat nog niet hebt gedaan, kunt u een [gratis proefversie aanvragen van een Azure-abonnement](https://azure.microsoft.com/pricing/free-trial/) en de [Azure CLI](../articles/xplat-cli-install.md) [ die met uw Azure-account is verbonden](../articles/xplat-cli-connect.md). Ga als volgt te werk om te controleren of de Azure-CLI zich in de modus Resource Manager bevindt:

```azurecli
azure config mode arm
```

Maak nu de schaalset met behulp van de opdracht `azure vmss quick-create`. In het volgende voorbeeld wordt een Linux-schaalset gemaakt genaamd `myVMSS` met 5 VM-exemplaren in de resourcegroep met de naam `myResourceGroup`:

```azurecli
azure vmss quick-create -n myVMSS -g myResourceGroup -l westus \
    -u ops -p P@ssw0rd! \
    -C 5 -Q Canonical:UbuntuServer:16.04.0-LTS:latest
```

In het volgende voorbeeld wordt een Windows-schaalset gemaakt met dezelfde configuratie:

```azurecli
azure vmss quick-create -n myVMSS -g myResourceGroup -l westus \
    -u ops -p P@ssw0rd! \
    -C 5 -Q MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest
```

Wilt u de locatie of image-urn aanpassen, bekijk dan de opdrachten `azure location list` en `azure vm image {list-publishers|list-offers|list-skus|list|show}`.

Wanneer deze opdracht wordt geretourneerd, zal de schaalset gemaakt zijn. Deze schaalset heeft een load balancer met een NAT-regelstoewijzingspoort 50, 000+i op de load balancer naar poort 22 op VM i. Zodra we de FQDN-NAAM van de load balancer achterhalen, zullen we hierdoor via ssh kunnen verbinden met onze VM's:

```bash
# (if you decide to run this as a script, please invoke using bash)

# list load balancers in the resource group we created
#
# generic syntax:
# azure network lb list -g RESOURCE-GROUP-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network lb list -g negatvmssrg | grep negatvmssrg)
split_line=( $line )
lb_name=${split_line[1]}

# now that we have the name of the load balancer, we can show the details to find which Public IP (PIP) is 
# associated to it
#
# generic syntax:
# azure network lb show -g RESOURCE-GROUP-NAME -n LOAD-BALANCER-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network lb show -g negatvmssrg -n $lb_name | grep loadBalancerFrontEnd)
split_line=( $line )
pip_name=${split_line[4]}

# now that we have the name of the public IP address, we can show the details to find the FQDN
#
# generic syntax:
# azure network public-ip show -g RESOURCE-GROUP-NAME -n PIP-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network public-ip show -g negatvmssrg -n $pip_name | grep FQDN)
split_line=( $line )
FQDN=${split_line[3]}

# now that we have the FQDN, we can use ssh on port 50,000+i to connect to VM i (where i is 0-indexed)
#
# example to connct via ssh into VM "0":
ssh -p 50000 negat@$FQDN
```

<!--HONumber=Dec16_HO1-->



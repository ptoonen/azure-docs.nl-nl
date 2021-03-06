---
title: Verbinding maken met VNets met het Resource Manager-implementatiemodel en Azure Portal | Microsoft Docs
description: Een VPN-gatewayverbinding maken tussen VNets met behulp van Resource Manager en Azure Portal.
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: carmonm
editor: 
tags: azure-resource-manager
ms.assetid: a7015cfc-764b-46a1-bfac-043d30a275df
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/25/2016
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: 28d81fe312195b9a9094e1ed066f5cba57c76933
ms.openlocfilehash: b85017913316a450fe19f1760abff6a86f933e2e


---
# <a name="configure-a-vnet-to-vnet-connection-using-the-azure-portal"></a>Een verbinding tussen VNets configureren met behulp van Azure Portal
> [!div class="op_single_selector"]
> * [Resource Manager - Azure Portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [Resource Manager - PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Klassiek - Klassieke portal](virtual-networks-configure-vnet-to-vnet-connection.md)
> 
> 

Dit artikel helpt u met de stappen voor het maken van een verbinding tussen VNets in het Resource Manager-implementatiemodel met behulp van VPN Gateway en Azure Portal.

Wanneer u Azure Portal gebruikt om virtuele netwerken te verbinden, moeten de VNets binnen hetzelfde abonnement vallen. Als uw virtuele netwerken onder verschillende abonnementen vallen, kunt u deze nog steeds verbinden met behulp van de [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)-stappen.

![v2v-diagram](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/v2vrmps.png)

### <a name="deployment-models-and-methods-for-vnet-to-vnet-connections"></a>Implementatiemodellen en -methoden voor verbindingen tussen VNets
[!INCLUDE [deployment models](../../includes/vpn-gateway-deployment-models-include.md)]

In de volgende tabel staan de momenteel beschikbare implementatiemodellen en -methoden voor VNet-naar-VNet-configuraties. Als er een artikel met configuratiestappen beschikbaar is, kunt u dit via een rechtstreekse koppeling in deze tabel raadplegen.

[!INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

**VNet-peering**

[!INCLUDE [vpn-gateway-vnetpeeringlink](../../includes/vpn-gateway-vnetpeeringlink-include.md)]

## <a name="about-vnet-to-vnet-connections"></a>Over VNet-naar-VNet-verbindingen
Het verbinden van een virtueel netwerk met een ander virtueel netwerk (VNet-naar-VNet) lijkt op het verbinden van een VNet met een on-premises locatie. Voor beide connectiviteitstypen wordt een Azure VPN-gateway gebruikt om een beveiligde tunnel met IPsec/IKE te bieden. De VNets die u verbindt, kunnen zich bevinden in verschillende regio's of tot verschillende abonnementen behoren.

U kunt zelfs VNet-naar-VNet-communicatie met multi-site-configuraties combineren. Zoals u in het volgende diagram kunt zien, kunt u netwerktopologieën maken waarin cross-premises connectiviteit wordt gecombineerd met connectiviteit tussen virtuele netwerken:

![Over verbindingen](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/aboutconnections.png "About connections")

### <a name="why-connect-virtual-networks"></a>Waarom virtuele netwerken koppelen?
U wilt virtuele netwerken wellicht koppelen om de volgende redenen:

* **Geografische redundantie en aanwezigheid tussen regio's**
  
  * U kunt uw eigen geo-replicatie of synchronisatie met beveiligde connectiviteit instellen zonder gebruik te maken van internetgerichte eindpunten.
  * Met Azure Traffic Manager en Load Balancer kunt u workloads met maximale beschikbaarheid instellen met behulp van geografische redundantie over meerdere Azure-regio's. Een belangrijk voorbeeld hiervan is het instellen van SQL Always On met beschikbaarheidsgroepen verspreid over meerdere Azure-regio's.
* **Regionale toepassingen met meerdere lagen met isolatie- of beheergrenzen**
  
  * Binnen dezelfde regio kunt u vanwege isolatie- of beheervereisten toepassingen met meerdere lagen instellen met meerdere virtuele netwerken die met elkaar zijn verbonden.

Zie voor meer informatie over verbindingen tussen VNets de [Veelgestelde vragen over VNet-naar-VNet](#faq) aan het einde van dit artikel.

### <a name="a-namevaluesaexample-settings"></a><a name="values"></a>Voorbeeldinstellingen
Wanneer u deze stappen uitvoert als oefening kunt u de volgende voorbeeldconfiguratiewaarden gebruiken. Als voorbeeld gebruiken we meerdere adresruimten voor elke VNet. Voor VNet-naar-VNet-configuraties zijn meerdere adresruimten echter niet vereist.

**Waarden voor TestVNet1:**

* VNet-naam: TestVNet1
* Adresruimte: 10.11.0.0/16
  * Subnetnaam: FrontEnd
  * Subnetadresbereik: 10.11.0.0/24
* Resourcegroep: TestRG1
* Locatie: VS - oost
* Adresruimte: 10.12.0.0/16
  * Subnetnaam: BackEnd
  * Subnetadresbereik: 10.12.0.0/24
* Naam gatewaysubnet: GatewaySubnet (dit wordt in de portal automatisch ingevuld)
  * Adresbereik gatewaysubnet: 10.11.255.0/27
* DNS-server: gebruik het IP-adres van de DNS-server
* Gatewaynaam van het virtuele netwerk: TestVNet1GW
* Gatewaytype: VPN
* VPN-type: op route gebaseerd
* KSU: selecteer de gateway-SKU die u wilt gebruiken
* Naam van openbare IP-adres: TestVNet1GWIP
* Verbindingswaarden:
  * Naam: TestVNet1toTestVNet4
  * Gedeelde sleutel: u kunt de gedeelde sleutel zelf maken. In dit voorbeeld gebruiken we abc123. Het belangrijkste is dat wanneer u de verbinding tussen de VNets maakt, de waarde moet overeenkomen.

**Waarden voor TestVNet4:**

* VNet-naam: TestVNet4
* Adresruimte: 10.41.0.0/16
  * Subnetnaam: FrontEnd
  * Subnetadresbereik: 10.41.0.0/24
* Resourcegroep: TestRG1
* Locatie: VS - west
* Adresruimte: 10.42.0.0/16
  * Subnetnaam: BackEnd
  * Subnetadresbereik: 10.42.0.0/24
* Naam GatewaySubnet: GatewaySubnet (dit wordt in de portal automatisch ingevuld)
  * Adresbereik GatewaySubnet: 10.41.255.0/27
* DNS-server: gebruik het IP-adres van de DNS-server
* Gatewaynaam van het virtuele netwerk: TestVNet4GW
* Gatewaytype: VPN
* VPN-type: op route gebaseerd
* KSU: selecteer de gateway-SKU die u wilt gebruiken
* Naam van openbare IP-adres: TestVNet4GWIP
* Verbindingswaarden:
  * Naam: TestVNet4toTestVNet1
  * Gedeelde sleutel: u kunt de gedeelde sleutel zelf maken. In dit voorbeeld gebruiken we abc123. Het belangrijkste is dat wanneer u de verbinding tussen de VNets maakt, de waarde moet overeenkomen.

## <a name="a-namecreatvneta1-create-and-configure-testvnet1"></a><a name="CreatVNet"></a>1. TestVNet1 maken en configureren
Als u al beschikt over een VNet, controleert u of de instellingen compatibel zijn met het ontwerp van de VPN-gateway. Let vooral op eventuele subnetten die met andere netwerken overlappen. Als u overlappende subnetten hebt, werkt de verbinding mogelijk niet goed. Als het VNet met de juiste instellingen is geconfigureerd, kunt u beginnen met de stappen in de sectie [Een DNS-server opgeven](#dns).

### <a name="to-create-a-virtual-network"></a>Een virtueel netwerk maken
[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="a-namesubnetsa2-add-additional-address-space-and-create-subnets"></a><a name="subnets"></a>2. Extra adresruimte toevoegen en subnetten maken
Wanneer het VNet is gemaakt, kunt u er extra adresruimte en subnetten aan toevoegen.

[!INCLUDE [vpn-gateway-additional-address-space](../../includes/vpn-gateway-additional-address-space-include.md)]

## <a name="a-namegatewaysubneta3-create-a-gateway-subnet"></a><a name="gatewaysubnet"></a>3. Een gatewaysubnet maken
Voordat u het virtuele netwerk verbindt met een gateway, moet u eerst het gatewaysubnet maken voor het virtuele netwerk waarmee u verbinding wilt maken. Indien mogelijk is het beste een gatewaysubnet met een CIDR-blok van /28 of /27 te gebruiken zodat er voldoende IP-adressen zijn om aan toekomstige aanvullende configuratievereisten te voldoen.

Als u deze configuratie bij wijze van oefening maakt, gebruikt u deze [voorbeeldinstellingen](#values) wanneer u het gatewaysubnet maakt.

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="to-create-a-gateway-subnet"></a>Een gatewaysubnet maken
[!INCLUDE [vpn-gateway-add-gwsubnet-rm-portal](../../includes/vpn-gateway-add-gwsubnet-rm-portal-include.md)]

## <a name="a-namednsservera4-specify-a-dns-server-optional"></a><a name="DNSServer"></a>4. Een DNS-server opgeven (optioneel)
Als u naamomzetting wilt instellen voor virtuele machines die in uw VNets worden geïmplementeerd, moet u een DNS-server opgeven.

[!INCLUDE [vpn-gateway-add-dns-rm-portal](../../includes/vpn-gateway-add-dns-rm-portal-include.md)]

## <a name="a-namevnetgatewaya5-create-a-virtual-network-gateway"></a><a name="VNetGateway"></a>5. De gateway van een virtueel netwerk maken
In deze stap maakt u de virtuele netwerkgateway VNet. Deze stap kan maximaal 45 minuten duren. Als u deze configuratie bij wijze van oefening maakt, kunt u de [voorbeeldinstellingen](#values) gebruiken.

### <a name="to-create-a-virtual-network-gateway"></a>De gateway van een virtueel netwerk maken
[!INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="a-namecreatetestvnet4a6-create-and-configure-testvnet4"></a><a name="CreateTestVNet4"></a>6. TestVNet4 maken en configureren
Wanneer u TestVNet1 hebt geconfigureerd, maakt u TestVNet4 door de vorige stappen te herhalen en de waarden te vervangen door die van TestVNet4. U hoeft niet te wachten tot de gateway van het virtuele netwerk voor TestVNet1 is gemaakt voordat u TestVNet4 configureert. Als u uw eigen waarden gebruikt, zorgt u ervoor dat de adresruimten niet overlappen met een van de VNets waarmee u verbinding wilt maken.

## <a name="a-nametestvnet1connectiona7-configure-the-testvnet1-connection"></a><a name="TestVNet1Connection"></a>7. De TestVNet1-verbinding configureren
Wanneer de virtuele netwerkgateways voor TestVNet1 en TestVNet4 zijn voltooid, kunt u de verbindingen voor uw virtuele netwerkgateway maken. In deze sectie maakt u een verbinding tussen VNet1 en VNet4.

1. In **Alle resources** gaat u naar de virtuele netwerkgateway voor uw VNet. Bijvoorbeeld **TestVNet1GW**. Klik op **TestVNet1GW** om de blade voor de virtuele netwerkgateway te openen.
   
    ![Blade Verbindingen](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/settings_connection.png "Connections blade")
2. Klik op **+Toevoegen** om de blade **Verbinding toevoegen** te openen.
3. Typ in het naamveld op de blade **Verbinding toevoegen** een naam voor de verbinding. Bijvoorbeeld **TestVNet1toTestVNet4**.
   
    ![Verbindingsnaam](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/v1tov4.png "Connection name")
4. Voor **Verbindingstype** selecteert u **VNet-naar-VNet** in de vervolgkeuzelijst.
5. De veldwaarde **Eerste virtuele netwerkgateway** wordt automatisch ingevuld omdat u deze verbinding maakt vanuit de opgegeven virtuele netwerkgateway.
6. Het veld **Tweede virtuele netwerkgateway** is de virtuele netwerkgateway van de VNet waarvoor u een verbinding wilt maken. Klik op **Een andere virtuele netwerkgateway kiezen** om de blade **Virtuele netwerkgateway kiezen** te openen.
   
    ![Verbinding toevoegen](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/add_connection.png "Add a connection")
7. Bekijk de virtuele netwerkgateways die worden vermeld op deze blade. U ziet dat er alleen virtuele netwerkgateways worden vermeld die binnen uw abonnement vallen. Als u verbinding wilt maken met een virtuele netwerkgateway die geen deel uitmaakt van uw abonnement, gebruikt u het [PowerShell-artikel](vpn-gateway-vnet-vnet-rm-ps.md). 
8. Klik op de virtuele netwerkgateway waarmee u verbinding wilt maken.
9. In het veld **Gedeelde sleutel** typt u een gedeelde sleutel voor de verbinding. U kunt deze sleutel ook zelf maken of genereren. In een verbinding tussen sites is de sleutel exact dezelfde voor het on-premises apparaat en uw virtuele netwerkgatewayverbinding. Het concept is hier vergelijkbaar, maar in plaats van een verbinding te maken met een VPN-apparaat, maakt u verbinding met een andere virtuele netwerkgateway.
   
    ![Gedeelde sleutel](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/sharedkey.png "Shared key")
10. Klik op **OK** onder aan de blade om de wijzigingen op te slaan.

## <a name="a-nametestvnet4connectiona8-configure-the-testvnet4-connection"></a><a name="TestVNet4Connection"></a>8. De TestVNet4-verbinding configureren
Maak vervolgens een verbinding tussen TestVNet4 en TestVNet1. Gebruik dezelfde methode die u hebt gebruikt voor het maken van de verbinding tussen TestVNet1 en TestVNet4. Zorg ervoor dat u dezelfde gedeelde sleutel gebruikt.

## <a name="a-nameverifyconnectiona9-verify-your-connection"></a><a name="VerifyConnection"></a>9. De verbinding controleren
Controleer de VPN-verbinding. Doe voor elke virtuele netwerkgateway het volgende:

1. Zoek de blade voor de gateway van het virtuele netwerk. Bijvoorbeeld **TestVNet4GW**. 
2. Klik op de blade van het virtuele netwerkgateway op **Verbindingen** om de blade Verbindingen weer te geven voor de virtuele netwerkgateway.

Geef de verbindingen weer en controleer de status. Nadat de verbinding is gemaakt, worden de statuswaarden weergegeven als **Geslaagd** en **Verbonden**.

![Geslaagd](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/connected.png "Succeeded")

U kunt afzonderlijk op elke verbinding dubbelklikken voor meer informatie over de verbinding.

![Essentials](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/essentials.png "Essentials")

## <a name="a-namefaqavnet-to-vnet-faq"></a><a name="faq"></a>Veelgestelde vragen over VNet-naar-VNet
Bekijk de Veelgestelde vragen voor meer informatie over VNet-naar-VNet-verbindingen.

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]

## <a name="next-steps"></a>Volgende stappen
Wanneer de verbinding is voltooid, kunt u virtuele machines aan uw virtuele netwerken toevoegen. Raadpleeg de [documentatie over Virtual Machines](https://docs.microsoft.com/azure/#pivot=services&panel=Compute) voor meer informatie.



<!--HONumber=Nov16_HO4-->



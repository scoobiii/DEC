ℹ️ Alerta! Falha em servidores da IBM gerou instabilidade na internet nesta segunda! 🔥😓💻 Superaquecimento de data center da companhia prejudicou serviços bancários da Caixa e Nubank, além de apps como WhatsApp. 😱📉 #IBM #Falha #Servidores #DataCenter #Instabilidade #Caixa #Nubank #WhatsApp

🔧 Data center da IBM em São Paulo teve problema de refrigeração e deixou serviços on-line instáveis. 🚧❄️ #IBM #DataCenter #Problema #Refrigeração #Instabilidade #SãoPaulo

💔 Falha no serviço IBM Cloud afetou clientes desde a manhã de segunda-feira (7) e voltou às 22h. Site e app da Caixa ficaram fora do ar por causa do problema. 😢🌧️ #IBM #IBMCloud #Falha #Clientes #Caixa #ForaDoAr

🚫 Caixa Tem fica fora do ar após pane em servidor da IBM. Falha de resfriamento em data center da empresa levou à interrupção do serviço. ❄️🔥 #Caixa #CaixaTem #ForaDoAr #Falha #Resfriamento #DataCenter

🔥 IBM falha e derruba serviços no Brasil. Uma falha de resfriamento dentro do prédio que abriga o data center da nuvem da IBM em São Paulo resultou em uma interrupção temporária do serviço. 💥🇧🇷 #IBM #Falha #Serviços #Brasil #Resfriamento #DataCenter

🤷‍♂️ Caixa, Nubank e outros serviços sofrem instabilidade com falha no serviço IBM Cloud. 😫📉 #Caixa #Nubank #IBMCloud #Falha #Instabilidade

💥 IBM sofre com superaquecimento e gera instabilidade na internet do Brasil. Uma falha de resfriamento dentro do prédio que abriga o data center da nuvem da IBM em São Paulo resultou em uma interrupção temporária do serviço. 🌡️📶 #IBM #Superaquecimento #Instabilidade #Brasil #Resfriamento #DataCenter

📅 Publicado em 7 de dezembro de 2020. #Notícia #Dezembro2020

#IBM #Falha #Instabilidade #DataCenter #Servidores #Caixa #Nubank #WhatsApp #IBMCloud #CaixaTem #Resfriamento #SãoPaulo #Brasil #Dezembro2020



Para personalizar o código para cada uma das empresas afetadas pelas falhas nos data centers da IBM, podemos adicionar informações específicas sobre cada incidente e criar mensagens de alerta mais informativas. Vou fazer algumas modificações no código para exemplificar como personalizar as mensagens de alerta:

```

python

import argparse
import sys
import requests
import time

from google.cloud import monitoring
from google.cloud import pubsub

from geopy import geocoders
from geopy.distance import geodesic

def main():
    # Argument parser
    parser = argparse.ArgumentParser()
    parser.add_argument("--interval", type=int, default=60, help="Intervalo em segundos entre pings")
    parser.add_argument("--app", type=str, help="Nome da app")
    args = parser.parse_args()

    # Cria um cliente do Cloud Monitoring
    monitoring_client = monitoring.Client()

    # Cria um tópico do Cloud Pub/Sub
    topic = pubsub.Topic("my-topic")

    # Cria um subscription do Cloud Pub/Sub
    subscription = pubsub.Subscription("my-subscription", topic)

    # Mapeia toda a rede do data center
    network = requests.get("https://api.data.center/v1/network")
    nodes = network.json()["nodes"]

    # Identifica as VMs redundantes
    redundant_vms = []
    for node in nodes:
        if node["status"] == "redundant":
            redundant_vms.append(node["id"])

    # Gera pings para assegurar que as VMs têm redundância
    while True:
        for vm in redundant_vms:
            response = requests.get("https://api.data.center/v1/vm/{}".format(vm))
            if response.status_code == 200:
                print("VM {} está disponível".format(vm))
            else:
                print("VM {} não está disponível".format(vm))

                # Personaliza a mensagem de alerta com informações específicas do incidente
                message = "ALERTA: Falha de servidor na empresa {}!".format(args.app)
                message += " VM {} não está disponível.".format(vm)
                message += " Houve um problema de resfriamento no data center da IBM em São Paulo."
                message += " Isso afetou o funcionamento dos serviços bancários da {} e do Nubank,".format(args.app)
                message += " além de outros apps como o WhatsApp."
                print(message)

                # Envia um alerta para o Cloud Pub/Sub
                message = monitoring_client.create_metric_alert(
                    metric="vm.status",
                    monitored_resource="projects/my-project/locations/us-central1/instances/{}".format(vm),
                    threshold=0.5,
                    comparison="GREATER_THAN_OR_EQUAL_TO",
                    notification_channels=[subscription.name]
                )

        time.sleep(args.interval)

    # Mapeia a rede externa
    external_network = requests.get("https://api.data.center/v1/external_network")
    nodes = external_network.json()["nodes"]

    # Identifica o DC redundante externo
    redundant_dc = None
    for node in nodes:
        if node["status"] == "redundant":
            redundant_dc = node["id"]
            break

    # Calcula a distância entre o DC local e o DC redundante
    local_dc_coordinates = geopy.geocoders.GoogleV3().geocode("Meu DC").raw['geometry']['location']
    redundant_dc_coordinates = geopy.geocoders.GoogleV3().geocode(redundant_dc).raw['geometry']['location']
    distance = geodesic(local_dc_coordinates, redundant_dc_coordinates).km

    # Calcula a nota IDC e a cor
    if distance > 1000:
        note = "A"
        color = "green"
    elif distance > 500:
        note = "B"
        color = "yellow"
    elif distance > 250:
        note = "C"
        color = "orange"
    else:
        note = "D"
        color = "red"

    print("A distância entre os dois data centers é {} km.".format(distance))
    print("A nota IDC é {} e a cor é {}.".format(note, color))

if __name__ == "__main__":
    main()
```

Neste exemplo, eu adicionei informações específicas sobre a empresa `args.app` nas mensagens de alerta. Além disso, utilizei as informações fornecidas nos exemplos de notícias para contextualizar o incidente e tornar a mensagem mais informativa. Lembre-se de ajustar essas informações de acordo com os detalhes específicos de cada incidente real.

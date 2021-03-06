---

copyright:
  years: 2015, 2018
lastupdated: "2018-05-07"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Señales de autenticación
Puede utilizar señales para escribir aplicaciones que hacen solicitudes autenticadas a servicios de {{site.data.keyword.ibmwatson}} sin incluir las credenciales de servicio en cada llamada.
{: shortdesc}

Puede escribir un proxy de autenticación en {{site.data.keyword.cloud}} que obtiene y devuelve una señal a la aplicación cliente, que podrá utilizar después la señal para llamar al servicio directamente. Este proxy elimina la necesidad de canalizar todas las solicitudes de servicio mediante una aplicación del lado del servidor intermedia, que será necesario evitar exponer las credenciales de servicio desde su aplicación cliente.

Para obtener más información sobre el modelo de programación de interacción directa, consulte [Programación de modelos para servicios de {{site.data.keyword.watson}}](/docs/services/watson/getting-started-develop.html).

Debe tener credenciales de autenticación básica HTTP para un servicio para obtener una señal para el servicio; para obtener más información, consulte [Credenciales de servicio para servicios de {{site.data.keyword.watson}}](/docs/services/watson/getting-started-credentials.html). Obtiene una señal para un servicio específico, y la señal solo funcionará para el servicio especificado.

## Obtención de una señal
Para obtener una señal para un servicio, envíe una solicitud `GET` HTTP al método `token` de la API de `autorización`. El host depende del servicio que esté utilizando. Puede identificar el host desde el punto final para la API de servicio.

- **gateway.watsonplatform.net**: Servicios como {{site.data.keyword.personalityinsightsshort}} invocan el método `token` en el punto final `https://gateway.watsonplatform.net/authorization/api/v1/token`
- **stream.watsonplatform.net**: Servicios como {{site.data.keyword.speechtotextshort}} y {{site.data.keyword.texttospeechshort}} invocan el método `token` en https://stream.watsonplatform.net/authorization/api/v1/token`

Para identificar el servicio para el que desee obtener una señal, pase el URL de base de servicio mediante el parámetro de consulta `url` del método. Por ejemplo, pase el valor siguiente para el parámetro `url` para captar una señal para el servicio de {{site.data.keyword.texttospeechshort}}:

```bash
url=https://stream.watsonplatform.net/text-to-speech/api
```
{: codeblock}

Pase las credenciales de autenticación básica HTTP para el servicio con la solicitud como lo haría para cualquier llamada que no utilice señales. Por ejemplo, el mandato curl siguiente obtiene una señal para el servicio de {{site.data.keyword.texttospeechshort}}:

```bash
curl -X GET --user {username}:{password} \
"https://stream.watsonplatform.net/authorization/api/v1/token?url=https://stream.watsonplatform.net/text-to-speech/api"
```
{: pre}

La opción `--user` pasa la concatenación del *username* y de *password* para el servicio, que puede obtener desde {{site.data.keyword.cloud_notm}} o desde la variable de entorno `VCAP_SERVICES` para una aplicación que esté vinculada a una instancia del servicio. Asegúrese de utilizar las credenciales de servicio, no su ID de inicio de sesión ni su contraseña de {{site.data.keyword.cloud_notm}}.

El método devuelve la señal como una codificación Base64 de un kilobyte de una carga útil cifrada.

Las señales tienen un tiempo de duración (TTL) de una hora, tras el cual no podrá utilizarlas para establecer una conexión con el servicio. Las conexiones existentes ya establecidas con la señal no están afectadas por el tiempo de espera excedido. Un intento de pasar una señal caducada o no válida obtiene un código de estado `401 Unauthorized` HTTP desde {{site.data.keyword.cloud_notm}}. El código de aplicación necesita prepararse para renovar la señal en respuesta a este código de retorno.

## Utilización de una señal

Pase una señal a la interfaz HTTP de un servicio utilizando la cabecera de solicitud `X-Watson-Authorization-Token`, el parámetro de consulta `watson-token`, o una cookie. Debe pasar la señal con cada solicitud HTTP.

Los siguientes ejemplos de curl demuestran los primeros dos enfoques.

- Obtenga una señal para el servicio de {{site.data.keyword.texttospeechshort}} y escríbala en un archivo con el nombre `token`.

  ```bash
  curl -X GET --user {username}:{password}
  --output token
  "https://stream.watsonplatform.net/authorization/api/v1/token?url=https://stream.watsonplatform.net/text-to-speech/api"
  ```
  {: pre}

- Pase el contenido del archivo `token` en una llamada al método `synthesize` de la API de {{site.data.keyword.texttospeechshort}}. El primer mandato pasa la señal mediante la cabecera `X-Watson-Authorization-Token`. El segundo mandato la pasa mediante el parámetro de consulta `watson-token`. Los dos mandatos son equivalentes. Cada mandato graba la salida de la llamada a un archivo denominado `hello_world.wav`.

    - *Desde un shell de UNIX o Linux,* redirija el contenido del archivo `token` al mandato:

      ```bash
      curl -X POST \
      --header "X-Watson-Authorization-Token: $(< ./token)" \
      --header "Content-Type: application/json" \
      --header "Accept: audio/wav" \
      --data "{\"text\":\"hello world\"}" \
      --output hello_world.wav \
      "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize"
      ```
      {: pre}

      ```bash
      curl -X POST \
      --header "Content-Type: application/json" \
      --header "Accept: audio/wav" \
      --data "{\"text\":\"hello world\"}" \
      --output hello_world.wav \
      "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize?watson-token=$(< ./token)"
      ```
      {: pre}

    - *Desde un indicador de mandatos de Windows,* incluya el contenido del archivo `token` en el mandato:

        ```bash
        curl -X POST
        --header "X-Watson-Authorization-Token: {token_string}"
        --header "Content-Type: application/json"
        --header "Accept: audio/wav"
        --data "{\"text\":\"hello world\"}"
        --output hello_world.wav
        "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize"
        ```
        {: pre}

        ```bash
        curl -X POST
        --header "Content-Type: application/json"
        --header "Accept: audio/wav"
        --data "{\"text\":\"hello world\"}"
        --output hello_world.wav
        "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize?watson-token={token_string}"
        ```
        {: pre}

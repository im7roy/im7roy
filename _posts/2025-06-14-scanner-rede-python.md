---
title: "Criando seu Primeiro Scanner de Rede em Python"
date: 2025-06-12
categories:
  - Cibersegurança
  - Redes
tags:
  - python
  - scanner
  - rede
---

Neste tutorial, vamos criar um scanner de rede básico em Python que identifica hosts ativos em uma rede local. Ferramentas como essa são essenciais para profissionais de segurança realizarem análises de infraestrutura.

## O Código Completo (Corrigido)

{% highlight python %}
import socket
import ipaddress
from concurrent.futures import ThreadPoolExecutor

def verificar_host(ip, portas=[22, 80, 443], timeout=1):
"""
Verifica se um host está ativo e portas abertas
"""
try: # Tentativa de conexão ICMP (ping)
socket.create_connection((str(ip), 80), timeout=timeout)
print(f"\n[+] Host ativo: {ip}")

        # Verificar portas
        for porta in portas:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(timeout)
            resultado = sock.connect_ex((str(ip), porta))
            if resultado == 0:
                print(f"   [+] Porta {porta} aberta")
            sock.close()
        return True
    except:
        return False

def scan_rede(rede_cidr, portas_alvo, max_threads=50):
"""
Escaneia toda uma sub-rede
"""
rede = ipaddress.ip_network(rede_cidr)
print(f"\n[!] Iniciando scan na rede: {rede}")

    with ThreadPoolExecutor(max_workers=max_threads) as executor:
        futures = [executor.submit(verificar_host, ip, portas_alvo) for ip in rede.hosts()]

        ativos = sum(1 for future in futures if future.result())

    print(f"\n[!] Scan completo! Hosts ativos: {ativos}/{len(list(rede.hosts()))}")

if **name** == "**main**": # Configurações
SUA_REDE = "192.168.1.1/24" # Altere para sua rede
PORTAS_ALVO = [21, 22, 23, 80, 443, 8080, 3389]

    scan_rede(SUA_REDE, PORTAS_ALVO)

{% endhighlight %}

## Explicação Passo a Passo

1. **Importações principais**

   - `socket`: cria conexões TCP e simula “ping” via tentativa de conexão à porta 80.
   - `ipaddress`: gera lista de hosts a partir de um CIDR.
   - `ThreadPoolExecutor`: paraleliza solicitações de rede para acelerar o scan.

2. **Função `verificar_host`**

   - Tenta conectar (TCP) à porta 80 para checar se o host está ativo.
   - Se ativo, itera sobre a lista de `portas` e usa `connect_ex` para testar cada porta.
   - Exibe no console quais portas estão abertas.

3. **Função `scan_rede`**

   - Constrói objeto `ip_network` a partir de `rede_cidr`.
   - Submete cada host ativo ao pool de threads, chamando `verificar_host`.
   - Conta quantos hosts retornaram `True` (ativos) e imprime resumo final.

4. **Bloco principal**
   - Define `SUA_REDE` (ex.: `"192.168.1.1/24"`) e `PORTAS_ALVO`.
   - Executa `scan_rede`, imprimindo progresso e resultados.

---

### Observações

- Este método “ping” (TCP-connect na porta 80) pode gerar falsos positivos em redes muito restritas.
- Para um scan ICMP real, use bibliotecas como `scapy` ou `icmplib` (requer privilégios de root).
- Ajuste `max_threads` para controlar o paralelismo e evitar sobrecarregar sua rede ou seu host.
- Em redes maiores (/24+), considere aumentar `timeout` ou reduzir `max_threads` para maior estabilidade.

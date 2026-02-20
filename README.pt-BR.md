# UFW-Cloudflare

🌐 [English](README.md) | **Português** | [Español](README.es.md) | [中文](README.zh.md) | [Íslenska](README.is.md)

Gerencia automaticamente os [ranges de IP da Cloudflare](https://www.cloudflare.com/ips/) no UFW (Uncomplicated Firewall). Este script busca os endereços IPv4 e IPv6 mais recentes da Cloudflare e cria regras de firewall para tráfego HTTP/HTTPS (portas 80 e 443 TCP), garantindo que apenas requisições proxied pela Cloudflare cheguem ao seu servidor de origem.

Desenvolvido para **servidores Ubuntu** (também compatível com sistemas baseados em Debian).

> **Prevenção contra DDoS:** Ao permitir apenas os ranges de IP da Cloudflare e bloquear todo o tráfego direto nas portas 80/443, seu servidor de origem se torna invisível para atacantes. Todas as requisições HTTP/HTTPS precisam passar pela rede da Cloudflare primeiro, aproveitando sua mitigação de DDoS, WAF e proteção contra bots antes de chegar ao seu servidor. Ataques DDoS diretos ao servidor de origem são efetivamente bloqueados, já que IPs fora da Cloudflare são negados no nível do firewall.

## Por quê?

Quando seu domínio é proxied pela Cloudflare, todo o tráfego dos visitantes chega a partir dos [endereços IP da Cloudflare](https://developers.cloudflare.com/fundamentals/concepts/cloudflare-ip-addresses/) em vez dos IPs individuais dos visitantes. Seu firewall precisa permitir esses ranges, caso contrário o tráfego legítimo será bloqueado. Sem essa configuração, o IP do seu servidor de origem pode ficar exposto e vulnerável a ataques DDoS diretos que contornam a Cloudflare. Este script automatiza esse processo e mantém as regras atualizadas.

## Requisitos

- **Ubuntu 20.04+** (ou sistema baseado em Debian)
- `ufw` instalado e habilitado
- `wget`
- Privilégios root (`sudo`)

## Início Rápido

```bash
wget -O ufw.sh https://raw.githubusercontent.com/hyzis-manager/ufw-cloudflare/main/ufw.sh
chmod +x ufw.sh
sudo ./ufw.sh
```

Na primeira execução, o script irá:

1. Buscar os ranges IPv4 e IPv6 atuais da Cloudflare em `cloudflare.com/ips-v4` e `cloudflare.com/ips-v6`
2. Criar regras de permissão no UFW para cada range nas portas **80** e **443** (TCP)
3. Perguntar se você deseja ativar o **Supervision** (atualização automática diária)

## Uso

```
sudo ./ufw.sh [opções]
```

| Opção | Curto | Descrição |
|---|---|---|
| `--help` | `-h` | Exibe a mensagem de ajuda |
| `--purge` | `-p` | Remove todas as regras existentes da Cloudflare (marcadas com comentário `#cloudflare`) antes de adicionar novas |
| `--no-new` | `-n` | Não busca IPs nem adiciona novas regras (use com `--purge` para apenas remover regras) |
| `--supervision` | `-s` | Ativa atualização automática diária via timer do systemd |

## Exemplos

**Primeira configuração** — buscar e permitir todos os IPs da Cloudflare:

```bash
sudo ./ufw.sh
```

**Atualizar regras** — remover regras antigas e adicionar as atuais:

```bash
sudo ./ufw.sh --purge
```

**Remover todas as regras da Cloudflare** sem adicionar novas:

```bash
sudo ./ufw.sh --purge --no-new
```

**Ativar atualização automática diária** (Supervision):

```bash
sudo ./ufw.sh --supervision
```

## Supervision

O recurso Supervision instala um **timer do systemd** que executa o script uma vez a cada 24 horas com `--purge`, garantindo que suas regras de firewall sempre reflitam os ranges de IP mais recentes da Cloudflare.

Quando ativado, duas units do systemd são criadas:

- `ufw-cloudflare-supervision.service` — serviço oneshot que executa o script
- `ufw-cloudflare-supervision.timer` — timer que dispara o serviço diariamente (e 5 minutos após o boot)

### Gerenciando o timer

```bash
# Verificar status do timer
systemctl status ufw-cloudflare-supervision.timer

# Ver próxima execução agendada
systemctl list-timers ufw-cloudflare-supervision.timer

# Desativar atualizações automáticas
sudo systemctl stop ufw-cloudflare-supervision.timer
sudo systemctl disable ufw-cloudflare-supervision.timer

# Disparar uma atualização manualmente
sudo systemctl start ufw-cloudflare-supervision.service
```

## Como Funciona

1. Busca os ranges IPv4 em `https://www.cloudflare.com/ips-v4`
2. Busca os ranges IPv6 em `https://www.cloudflare.com/ips-v6`
3. Para cada range CIDR, executa: `ufw allow from <IP> to any port 80,443 proto tcp comment "cloudflare"`
4. Quando `--purge` é usado, remove todas as regras UFW marcadas com o comentário `# cloudflare` antes de adicionar novas
5. Valida os formatos de IP e verifica se o download foi bem-sucedido antes de aplicar alterações

## Legenda da Saída

Durante a execução, o script exibe indicadores de progresso:

- **`+`** (verde) — regra criada
- **`-`** (vermelho) — regra excluída
- **`.`** (cinza) — regra ignorada (já existe ou inválida)

## Ranges de IP da Cloudflare

O script busca os IPs diretamente dos endpoints oficiais da Cloudflare. Esses ranges também estão disponíveis via [API da Cloudflare](https://developers.cloudflare.com/api/resources/ips/methods/list/) em `https://api.cloudflare.com/client/v4/ips` (sem necessidade de autenticação). A Cloudflare atualiza esses ranges com pouca frequência, mas recomenda manter sua allowlist atualizada.

## Licença

MIT

# Portainer Agent: Implantação Padronizada para Hosts Docker

## 🎯 Visão Geral

Este projeto implanta o **Portainer Agent** em um host Docker. O agente atua como um cliente leve que se comunica com uma instância central do **Portainer Server**, permitindo o gerenciamento de múltiplos ambientes Docker a partir de uma única interface.

O objetivo deste repositório é fornecer um método padronizado e repetível para implantar o agente, utilizando um `Makefile` para simplificar o ciclo de vida da aplicação.

**Nota Importante:** Este componente deve ser implantado em cada host Docker que você deseja gerenciar remotamente. Ele não substitui o `Portainer Server`.

## 🏗️ Arquitetura e Pontos de Atenção

A simplicidade deste `docker-compose.yaml` é intencional, mas exige atenção a pontos críticos de arquitetura e segurança.

1. **Comunicação Direta:** Diferente dos nossos outros serviços, o Portainer Agent **não é exposto via Traefik**. Ele expõe sua porta de comunicação (padrão `9001`) diretamente no host. Isso ocorre porque ele não é um serviço web para usuários, e sim um endpoint de gerenciamento para o Portainer Server.

      * **Recomendação:** Em um ambiente de produção, a porta do agente (`9001`) deve ser protegida por regras de firewall, permitindo o acesso **apenas** a partir do IP do seu Portainer Server.

2. **⚠️ ALERTA DE SEGURANÇA: Acesso irrestrito ao Host**
    O `docker-compose.yaml` inclui o volume `- /:/host`. É fundamental entender o que isso significa:

      * **O quê:** Este comando monta **todo o sistema de arquivos raiz do host** dentro do contêiner do agente.
      * **Por quê:** O Portainer utiliza esse acesso para funcionalidades avançadas, como a navegação no sistema de arquivos do host (`host-management/browse`) diretamente da UI.
      * **O Risco:** Isso concede ao contêiner do agente privilégios equivalentes aos de `root` no host. Se o contêiner do agente for comprometido, o atacante terá acesso irrestrito a todo o seu servidor.

    **Decisão de Trade-off:** Ao implantar este agente, você está fazendo uma troca consciente entre funcionalidade (capacidade de gerenciar o host via UI) e segurança. Para ambientes de alta segurança, avalie se essa funcionalidade é estritamente necessária.

3. **Acesso ao Socket do Docker:** O agente precisa de acesso ao socket do Docker (`/var/run/docker.sock`) e ao diretório de volumes (`/var/lib/docker/volumes`) para poder gerenciar os contêineres e volumes do host. Esta é uma prática padrão para ferramentas de gerenciamento Docker.

## ✅ Pré-requisitos

* Docker Engine e Docker Compose.
* Um shell compatível com `bash`.
* Uma instância do `Portainer Server` rodando e acessível pela rede.

## 🚀 Configuração e Deploy

### 1\. Clone o Repositório

```bash
git clone https://github.com/RafaelQSantos-RQS/portainer-agent.git
cd portainer-agent
```

### 2\. Prepare o Ambiente

Execute o comando de setup para gerar o arquivo de configuração de ambiente.

```bash
make setup
```

Este comando criará um arquivo `.env` a partir do template (`.env.template`) na primeira execução.

### 3\. Configure as Variáveis

Edite o arquivo `.env` com os valores para o seu ambiente:

* `AGENT_VERSION`: Fixe uma versão estável da imagem.
* `AGENT_PORT`: A porta que o agente irá expor no host. Mantenha `9001` a menos que haja um conflito.

### 4\. Inicie o Agente

```bash
make up
```

O contêiner do agente será iniciado e estará pronto para ser adicionado ao seu Portainer Server.

## 🔗 Integração com o Portainer Server

Com o agente rodando no seu host remoto, siga estes passos na interface do seu **Portainer Server**:

1. Navegue até **Environments** (ou Endpoints).
2. Clique em **Add environment**.
3. Selecione a opção **Docker Standalone**.
4. Clique em **Start Wizard**.
5. Selecione a opção **Agent**.
6. Preencha o nome do ambiente (ex: `docker-host-01`) e o endereço do agente no formato `IP_DO_HOST:PORTA` (ex: `192.168.1.10:9001`).
7. Clique em **Connect**.

Se tudo estiver correto, seu novo ambiente aparecerá na lista e poderá ser gerenciado centralmente.

## 🧰 Uso e Manutenção (Comandos do Makefile)

A gestão do agente é feita via `Makefile`.

```bash
# Mostra todos os comandos disponíveis
make help

# Prepara o ambiente (cria .env se necessário)
make setup

# Sobe o contêiner
make up

# Para e remove o contêiner
make down

# Reinicia o serviço
make restart

# Acompanha os logs em tempo real
make logs
```

**⚠️ Aviso:** O comando `make sync` irá descartar todas as suas alterações locais e forçar a sincronia com o repositório remoto. Use com extremo cuidado.

## 💬 Contato e Contribuições

Este projeto é mantido como parte do meu portfólio pessoal. Caso encontre problemas ou tenha sugestões, por favor, abra uma **Issue** neste repositório. Para outros assuntos, encontre-me no [LinkedIn](https://www.linkedin.com/in/rafael-queiroz-santos).

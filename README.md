# Ansible Vault Setup

## Descrição

Este projeto Ansible automatiza a instalação e configuração do HashiCorp Vault com backend Raft. Ele provisiona um servidor Vault, configura o armazenamento Raft, cria usuários e grupos necessários e configura o serviço systemd para gerenciamento.

## Pré-requisitos

- Ansible >= 2.9
- Python 3 instalado no host de controle Ansible
- Acesso SSH ao servidor Vault com um usuário que pode executar comandos `sudo`
- HashiCorp Vault >= 1.19.0

## Componentes

- **inventory.ini**: Define os servidores Vault para provisionamento.
- **playbook.yml**: Orquestra a instalação e configuração do Vault.
- **roles/vault**: Contém as tarefas, variáveis e templates para configurar o Vault.
  - **tasks/main.yml**: Lógica principal para instalar e configurar o Vault.
  - **vars/main.yml**: Variáveis padrão para a instalação do Vault.
  - **templates/vault.hcl.j2**: Template para o arquivo de configuração do Vault.
  - **templates/vault.service.j2**: Template para o arquivo de serviço systemd do Vault.

## Instalação

1.  Clone este repositório:

    ```bash
    git clone <seu_repositorio_url>
    cd ansible-vault
    ```

2.  Configure o arquivo `inventory.ini` com os detalhes do seu servidor Vault. Certifique-se de fornecer o `ansible_host`, `ansible_user` e `ansible_ssh_private_key_file` corretos.

    ```ini
    [vault_servers]
    vault-01 ansible_host=192.168.31.106 ansible_user=caraujo ansible_ssh_private_key_file=/home/carlosaraujo/.ssh/chave_carlos
    ```

3.  Instale as dependências Ansible (se necessário):

    ```bash
    ansible-galaxy collection install ansible.builtin
    ```

## Como usar

1.  Execute o playbook Ansible para instalar e configurar o Vault:

    ```bash
    ansible-playbook -i inventory.ini playbook.yml -b
    ```

    O `-b` flag executa as tarefas com privilégios de administrador (become).

2.  Após a conclusão bem-sucedida, o Vault estará instalado e configurado no servidor especificado.

## Inicialização do Vault

O playbook inclui uma tarefa para inicializar o Vault. As chaves de unseal e o token root são salvos localmente no arquivo `vault-init.json`. **Certifique-se de proteger este arquivo e excluí-lo do controle de versão.**

**Aviso:** Este setup usa `key-shares=1` e `key-threshold=1` para simplificar. Em produção, use valores mais altos para segurança.

## Exemplos de comandos

1.  Verifique o status do serviço Vault:

    ```bash
    systemctl status vault
    ```

2.  Acesse a interface web do Vault:

    Abra um navegador e acesse `http://<seu_servidor>:8200`.

3.  Autentique-se usando o token root do arquivo `vault-init.json`.

4.  Deslacrar o Vault usando a chave de unseal do arquivo `vault-init.json`.

## Arquitetura

```
+---------------------+     +----------------------+     +-----------------------+
| Controle Ansible    |---->| Servidor Vault      |---->| Backend de Armazen.   |
| (playbook.yml)      |     | (vault.service,     |     | (Raft)                |
|                     |     | vault.hcl)          |     |                       |
+---------------------+     +----------------------+     +-----------------------+
```

O diagrama acima ilustra a arquitetura básica da implementação:

1. **Controle Ansible**: Executa o playbook que configura o servidor
2. **Servidor Vault**: Executa o serviço Vault configurado via systemd
3. **Backend de Armazenamento**: Utiliza o modo Raft para armazenamento integrado
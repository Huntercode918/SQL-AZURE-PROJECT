# SQL-AZURE-PROJECT
# Bem-vindo ao Guia Rápido de Instância Gerenciada de SQL no Azure

Este `README.md` guia você, de forma didática e completa, pelo processo de criar e configurar uma Instância Gerenciada de SQL do Azure. Inclui tabelas resumindo cada etapa, exemplos de código (Portal, PowerShell e CLI) e dicas práticas.

---

## Sumário

1. [Visão Geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Passo 1: Criar a Instância](#passo-1-criar-a-instância)

   * [Portal do Azure](#portal-do-azure)
   * [PowerShell](#powershell)
   * [Azure CLI](#azure-cli)
4. [Passo 2: Revisar Rede e Segurança](#passo-2-revisar-rede-e-segurança)
5. [Passo 3: Criar Banco de Dados](#passo-3-criar-banco-de-dados)
6. [Passo 4: Recuperar Conexão](#passo-4-recuperar-conexão)
7. [Próximos Passos](#próximos-passos)

---

## Visão Geral

A Instância Gerenciada de SQL do Azure (Managed Instance) oferece compatibilidade quase total com o SQL Server on‑premises, permitindo migração simples de bancos de dados críticos sem reescrever aplicações.

**Principais benefícios**:

* Compatibilidade avançada com T‑SQL e recursos de SQL Server
* Backup gerenciado e alta disponibilidade nativa
* Escalabilidade de compute e storage independentes

---

## Pré-requisitos

| Item                    | Descrição                                                            |
| ----------------------- | -------------------------------------------------------------------- |
| **Assinatura do Azure** | Conta ativa (ou conta [gratuita](https://azure.microsoft.com/free/)) |
| **Az.SQL (PowerShell)** | Módulo `Az.SQL` atualizado (use `Update-Module Az.SQL`)              |
| **Azure CLI**           | CLI >= 2.0 (use `az version`)                                        |
| **Rede**                | VNet e sub-rede delegada à Managed Instance                          |

---

## Passo 1: Criar a Instância

### Portal do Azure

1. Acesse o [Portal do Azure](https://portal.azure.com).

2. Navegue em **SQL do Azure** > **+ Criar** > **Instância Gerenciada de SQL**.

3. Guia **Básico**: preencha conforme tabela:

   | Configuração      | Exemplo de Valor      | Observações                             |
   | ----------------- | --------------------- | --------------------------------------- |
   | Assinatura        | MinhaAssinatura       | Permissão de Proprietário               |
   | Grupo de Recursos | RG-SQLMI              | Deve seguir regras de nomenclatura      |
   | Nome da Instância | sqlmi-quickstart      | Evite nomes reservados                  |
   | Região            | eastus                | Escolha próxima aos clientes            |
   | Autenticação      | SQL                   | Pode usar também Azure AD               |
   | Admin / Senha     | adminsql / Senha!2025 | Mínimo 16 caracteres, complexidade alta |

4. Guia **Computação + armazenamento**:

   | Configuração        | Valor                | Descrição                      |
   | ------------------- | -------------------- | ------------------------------ |
   | Camada de Serviço   | General Purpose (GP) | Ideal para a maioria dos casos |
   | Geração de Hardware | Gen5                 | Standard, estável              |
   | vCores              | 8                    | Ajuste conforme carga          |
   | Armazenamento (GB)  | 512                  | Escalável depois               |
   | Licenciamento       | Pay-as-you-go        | Ou Azure Hybrid Benefit        |
   | Backup              | Geo-redundant        | Garantia de recuperação        |

5. Guia **Rede**:

   * Selecione VNet/Sub‑rede
   * Desabilite endpoint público (recomendado)
   * Configure NSG para conexões internas

6. (Opcional) **Configurações adicionais** e **Marcas**: defina collation, fuso horário, janelas de manutenção e tags como `Owner` e `Environment`.

7. Clique em **Revisar + criar** > **Criar**. Aguarde a implantação (\~30–60 min).

### PowerShell

```powershell
Connect-AzAccount

$subnetId = "/subscriptions/0000-0000-0000-0000/resourceGroups/RG-SQLMI/providers/Microsoft.Network/virtualNetworks/vnet01/subnets/sqlmi"

New-AzSqlInstance \
  -ResourceGroupName "RG-SQLMI" \
  -Name "sqlmi-quickstart" \
  -Location "eastus" \
  -SkuName "GP_Gen5_8" \
  -AdministratorLogin "adminsql" \
  -AdministratorLoginPassword (ConvertTo-SecureString "Senha!Complexa2025" -AsPlainText -Force) \
  -SubnetId $subnetId
```

### Azure CLI

```bash
az login

SUBNET_ID="/subscriptions/0000-0000-0000-0000/resourceGroups/RG-SQLMI/providers/Microsoft.Network/virtualNetworks/vnet01/subnets/sqlmi"

az sql mi create \
  --name sqlmi-quickstart \
  --resource-group RG-SQLMI \
  --location eastus \
  --admin-user adminsql \
  --admin-password "Senha!Complexa2025" \
  --subnet $SUBNET_ID \
  --sku-name GP_Gen5_8
```

---

## Passo 2: Revisar Rede e Segurança

| Item                         | Ação                                                              |
| ---------------------------- | ----------------------------------------------------------------- |
| **Tabela de Rotas**          | Verifique rotas UDR para tráfego MI                               |
| **NSG**                      | Confirme regras de entrada/saída para porta 1433                  |
| **Ponto Público (opcional)** | Abra portas conforme necessidade (1433 e 3342 para administração) |

---

## Passo 3: Criar Banco de Dados

### Portal do Azure

1. Acesse sua MI no Portal.
2. Em **Visão Geral**, clique em **+ Novo banco de dados**.
3. Guia **Básico**:

   * Nome: `db_quickstart`
   * Fonte: `None` (banco vazio)
4. Clique em **Revisar + criar** > **Criar**.

### PowerShell

```powershell
New-AzSqlDatabase \
  -ResourceGroupName "RG-SQLMI" \
  -InstanceName "sqlmi-quickstart" \
  -DatabaseName "db_quickstart" \
  -RequestedServiceObjectiveName "GP_Gen5_2"
```

### Azure CLI

```bash
az sql db create \
  --resource-group RG-SQLMI \
  --mi sqlmi-quickstart \
  --name db_quickstart \
  --service-objective GP_Gen5_2
```

---

## Passo 4: Recuperar Conexão

1. No Portal, abra sua MI.
2. Na guia **Visão Geral**, copie o **Host** (FQDN).
3. String de conexão:

```ini
Server=<seu-host>.database.windows.net;Database=db_quickstart;User Id=adminsql;Password=Senha!Complexa2025;
```

---

## Próximos Passos

* Conectar aplicações a partir de clientes ODBC/JDBC
* Monitorar performance com Azure Monitor e Query Store
* Automatizar backups e segurança com Azure Policy
* Planejar migração de bancos on‑premises usando Data Migration Service

# VPC Module

Este módulo cria uma infraestrutura de VPC na AWS com subnets públicas, privadas e isoladas.

## Recursos Criados

### Rede
- **VPC** - Virtual Private Cloud com CIDR 10.0.0.0/16
- **Internet Gateway** - Permite acesso à internet para subnets públicas
- **NAT Gateway** - Permite acesso à internet para subnets privadas (saída)

### Subnets
- **2 Public Subnets** - Subnets com acesso à internet (us-east-1a, us-east-1b)
- **2 Private Subnets** - Subnets com acesso à internet via NAT (us-east-1a, us-east-1b)
- **2 Isolated Subnets** - Subnets sem acesso à internet (desabilitadas por padrão)

### Segurança
- **Security Group** - Grupo de segurança "web" com regras de ingresso

## Técnicas Utilizadas

### count

Utilizado para criar múltiplos recursos idênticos baseado em uma **lista**.

**Definição no locals:**
```hcl
locals {
  azs            = ["us-east-1a", "us-east-1b"]
  public_subnets = ["10.0.0.0/19", "10.0.32.0/19"]
}
```

**Utilização no recurso:**
```hcl
resource "aws_subnet" "public" {
  count = length(local.public_subnets)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = local.public_subnets[count.index]
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${local.env}-${local.azs[count.index]}"
  }
}
```

---

### for_each

Utilizado para criar recursos baseados em um **mapa (object)**. Ideal quando cada recurso precisa de valores distintos.

**Definição no locals:**
```hcl
locals {
  vpc_cidr = "10.0.0.0/16"
  
  private_subnets = {
    private_1 = {
      cidr = cidrsubnet(local.vpc_cidr, 3, 2)
      az   = "us-east-1a"
    }
    private_2 = {
      cidr = cidrsubnet(local.vpc_cidr, 3, 3)
      az   = "us-east-1b"
    }
  }
}
```

**Utilização no recurso:**
```hcl
resource "aws_subnet" "private" {
  for_each = local.private_subnets

  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az

  tags = {
    Name = "${local.env}-private-${each.value.az}"
  }
}
```

**Acesso:** `aws_subnet.private["private_1"].id`

---

### cidrsubnet

Função nativa do Terraform para calcular subnets filhas automaticamente a partir de um CIDR base.

**Sintaxe:**
```hcl
cidrsubnet(cidr_block, new_bits, subnet_num)
```

- `cidr_block`: CIDR base (ex: "10.0.0.0/16")
- `new_bits`: número de bits para adicionar ao prefixo (ex: 3 = /19)
- `subnet_num`: número da subnet (ex: 2, 3, 4...)

**Exemplo:**
```hcl
cidrsubnet("10.0.0.0/16", 3, 2)  # Resultado: "10.0.64.0/19"
cidrsubnet("10.0.0.0/16", 3, 3)  # Resultado: "10.0.96.0/19"
```

---

### count condicional

Utilizado para criar recursos condicionalmente baseado em uma expressão ternária.

**Definição no locals:**
```hcl
locals {
  create_isolated_subnets = true
}
```

**Utilização no recurso:**
```hcl
resource "aws_subnet" "isolated_zone1" {
  count = local.create_isolated_subnets ? 1 : 0

  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.128.0/19"
  availability_zone = "us-east-1a"

  tags = {
    Name = "dev-isolated-us-east-1a"
  }
}
```

---

### dynamic

Utilizado para criar blocos dinamicamente dentro de recursos. Ideal para regras de firewall ou entradas em listas.

**Definição no locals:**
```hcl
locals {
  ingress_rules = {
    22 = ["63.10.10.10/32", "8.8.8.8/32"]  # porta 22
    80 = ["0.0.0.0/0"]                      # porta 80
  }
}
```

**Utilização no recurso:**
```hcl
resource "aws_security_group" "web" {
  name   = "web"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = local.ingress_rules

    content {
      from_port   = ingress.key
      to_port     = ingress.key
      protocol    = "tcp"
      cidr_blocks = ingress.value
    }
  }
}
```

---

### locals

Bloco que define variáveis reutilizáveis em todo o módulo. Suporta:
- **Listas**: `["a", "b", "c"]`
- **Mapas**: `{ key1 = "value1", key2 = "value2" }`
- **Objetos**: `{ key = { attr1 = "value", attr2 = "value" } }`
- **Funções**: `cidrsubnet()`, `length()`, etc.

## Variáveis Locais

| Variável | Descrição | Valor Padrão |
|---------|-----------|-------------|
| region | Região AWS | us-east-1 |
| vpc_cidr | CIDR da VPC | 10.75.0.0/16 |
| env | Ambiente | dev |
| azs | Zonas de disponibilidade | ["us-east-1a", "us-east-1b"] |
| public_subnets | CIDRs das subnets públicas | ["10.75.0.0/19", "10.75.32.0/19"] |
| private_subnets | Mapa de subnets privadas | { public_1 = {...}, public_2 = {...} } |
| create_isolated_subnets | Criar subnets isoladas | true |
| ingress_rules | Regras de ingresso do SG | { 22 = [...], 80 = [...] } |
## O problema em detalhe

- A pen (58 GB) aparecia no Windows **sem letra de unidade** e sem dar para
  formatar à maneira simples.
- Na **Gestão de Discos** via-se o disco "picado" em vários pedaços pequenos
  (partições de 4 MB, etc.) e muito espaço "Não Atribuído" — a marca típica de
  uma pen que levou com uma imagem de arranque Linux.

## Tentativa pelo Windows

Tentei resolver pela **Gestão de Discos** (`diskmgmt.msc`), eliminando volume a
volume. Funciona, mas é lento, confuso e cheio de passos — tinha de carregar
bloco a bloco com o botão direito, e era fácil enganar-me no disco. Sem muito
sucesso.

## A solução: Kali Linux

Em vez de lutar com o Windows, fui ao Kali e resolvi em **dois comandos**.

**1. Confirmar qual é a pen** (passo crítico — nunca limpar o disco errado):

```bash
lsblk
```

Identifiquei a pen pelo tamanho: `sdb`, 58,2 GB.

**2. Apagar as marcas das partições (assinaturas):**

```bash
sudo wipefs -a /dev/sdb
```

**3. Criar um novo sistema de ficheiros exFAT** (lido por Windows e Linux):

```bash
sudo mkfs.exfat /dev/sdb
```

**4. Ejetar em segurança:**

```bash
sudo eject /dev/sdb
```

Resultado: a pen ficou **limpa e formatada em segundos**. No Windows, bastou
atribuir-lhe uma letra de unidade (Gestão de Discos → *Alterar a Letra de
Unidade e Caminhos* → *Adicionar*) e passou a aparecer normalmente.

## Bónus: transferir ficheiros mais depressa com o Ubuntu no WSL

Para repor os dados na pen, usei o **Ubuntu no WSL**. As unidades do Windows
aparecem em `/mnt`, mas como a pen foi metida depois de o Ubuntu já estar aberto,
tive de a montar à mão:

```bash
sudo mkdir /mnt/d
sudo mount -t drvfs D: /mnt/d
```

Depois copiei a pasta inteira para a pen:

```bash
cp -r "/mnt/c/Users/Josia/Music/pen drive" /mnt/d/
```

> Nota: `-r` copia a pasta com todo o conteúdo. Aspas por causa do espaço no
> nome da pasta. As pastas do Windows aparecem no WSL em `/mnt/c/...`.

## Conclusão

Onde o Windows tropeçou, a **linha de comandos do Linux** resolveu em segundos.
Ferramentas como `wipefs` e `mkfs.exfat` dão acesso direto ao disco, sem as
camadas que o Windows mete pelo meio. Lição prática: para gerir discos e
sistemas de ficheiros, a CLI é rápida, direta e faz exatamente o que se manda.

> Ressalva honesta: na cópia de ficheiros, a velocidade é limitada sobretudo
> pela **própria pen**. O Linux costuma copiar com menos peso à volta, mas se a
> pen for lenta a escrever, nem o Linux faz milagres — e o Kali não seria mais
> rápido que o Ubuntu nisto, porque ambos usam o mesmo motor por baixo no WSL.

## Comandos usados

| Comando | O que faz |
|---|---|
| `lsblk` | Lista discos e partições (identificar a pen) |
| `sudo wipefs -a /dev/sdb` | Apaga as assinaturas das partições |
| `sudo mkfs.exfat /dev/sdb` | Cria sistema de ficheiros exFAT |
| `sudo eject /dev/sdb` | Ejeta a pen em segurança |
| `sudo mount -t drvfs D: /mnt/d` | Monta a unidade D do Windows no WSL |
| `cp -r "origem" /mnt/d/` | Copia uma pasta inteira para a pen |

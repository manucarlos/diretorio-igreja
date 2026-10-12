# Diretório de Serviços da Igreja

Site estático onde os membros da Vertical Church encontram produtos e serviços oferecidos por outros membros (cabeleireiros, pedreiros, boleiras, informática, saúde, educação...). O visitante escolhe uma categoria, vê os empreendedores e, para ver o WhatsApp de um deles, deixa o próprio nome e WhatsApp.

Não há servidor próprio nem etapa de build: são duas páginas HTML com CSS e JavaScript embutidos. Os dados vêm de um **Google Apps Script** publicado como aplicativo da web (ver [Como funciona](#como-funciona)).

## Arquivos

| Arquivo | O que é |
| --- | --- |
| `index.html` | Página inicial: lista de categorias (cartões com ícone) e, ao clicar, um modal com os empreendedores da categoria |
| `empreendedor.html` | Detalhes de um empreendedor (`empreendedor.html?id=<id>`): sobre, preço, site, Instagram e o formulário que libera o WhatsApp |

## Como funciona

```
Navegador  ──GET/POST──▶  Google Apps Script (web app)  ──▶  planilha com os dados
 index.html                  SCRIPT_URL (constante no topo
 empreendedor.html           do <script> de cada página)
```

A constante `SCRIPT_URL`, no início do `<script>` de **cada** página, aponta para o aplicativo da web. Ele responde JSON. As ações que as páginas usam:

| Ação | Método | Parâmetros | Resposta esperada | Usada em |
| --- | --- | --- | --- | --- |
| `getCategories` | GET | `action` | `{ categories: [{ id, name }] }` | `index.html` |
| `getEntrepreneursByCategory` | GET | `action`, `categoryId` | `{ entrepreneurs: [{ id, name, logo, prices, description }] }` | `index.html` |
| `getEntrepreneur` | GET | `action`, `id` | `{ entrepreneur: { id, name, category, logo, description, prices, website, instagram } }` | `empreendedor.html` |
| `saveUserData` | POST (JSON) | `action`, `name`, `whatsapp`, `entrepreneurId`, `entrepreneurName` | `{ success: true, whatsappNumber }` | `empreendedor.html` |

Em qualquer ação, `{ error: "mensagem" }` faz a página mostrar um alerta vermelho com o texto.

### O fluxo do WhatsApp

O número do empreendedor **não vem** em `getEntrepreneur`. Só é devolvido por `saveUserData`, depois que o visitante informa o próprio nome e WhatsApp. É assim que o diretório sabe quem procurou quem. A página então mostra o número formatado (`+55 (11) 91234-5678`) e um botão para `https://wa.me/<número>`.

### Ícones das categorias

O mapa `categoryIcons`, em `index.html`, liga o `id` da categoria a um ícone do Font Awesome. Categoria sem entrada no mapa usa o ícone padrão (loja):

| `id` da categoria | Ícone |
| --- | --- |
| `cabeleireiros` | tesoura |
| `pedreiros` | capacete |
| `boleiras` | bolo |
| `doces-e-guloseimas` | bengala de doce |
| `costura-e-artesanato` | camiseta |
| `informatica` | notebook |
| `saude-e-bem-estar`, `psicologos` | coração com pulso |
| `educacao` | livro |
| qualquer outra | loja |

Para criar uma categoria nova, cadastre-a na planilha; se quiser um ícone próprio, acrescente a linha no mapa.

## Dependências

Todas por CDN, sem `npm install`:

- Bootstrap 5.3.0-alpha1 (CSS e JavaScript)
- Font Awesome 6.0.0-beta3

## Rodar e publicar

Abra o `index.html` no navegador (ou sirva a pasta com qualquer servidor estático, por exemplo `python -m http.server`). Funciona também no GitHub Pages: basta publicar a pasta raiz da branch `main`.

Para trocar o backend, mude `SCRIPT_URL` nos dois arquivos. O Apps Script precisa estar publicado como aplicativo da web com acesso para "qualquer pessoa", senão o navegador recebe erro de CORS.

## Identidade visual

As cores ficam em variáveis CSS no topo de cada página (`:root`):

| Variável | Valor | Uso |
| --- | --- | --- |
| `--primary-color` | `#3a5a97` | cabeçalho, rodapé, botões |
| `--accent-color` | `#e3a04b` | destaques |
| `--secondary-color` | `#f8f9fa` | fundos claros |
| `--text-color` | `#333` | texto |

O mesmo bloco de estilos está **duplicado** nas duas páginas; ao mudar uma cor, mude nas duas.

## Limitações conhecidas

Descritas aqui para quem for mexer no código. Nenhuma foi alterada nesta documentação.

- **O backend não está neste repositório.** O código do Apps Script e a planilha ficam na conta Google de quem administra o diretório. Sem acesso a ela, não dá para mudar o formato das respostas.
- **Texto vindo do servidor entra na página sem escapar** (`innerHTML` com nome, descrição, preço, logo e site). Se alguém cadastrar `<script>` ou `javascript:` num campo da planilha, ele roda no navegador dos visitantes. Quem cadastra os empreendedores é de confiança hoje; se o cadastro passar a ser aberto, escape os campos ou use `textContent`.
- **Dados pessoais de visitantes.** `saveUserData` guarda nome e WhatsApp de quem procurou um empreendedor. Isso cai na LGPD: a página não informa para quê os dados são usados nem pede consentimento. Vale um aviso curto acima do formulário e uma política de quem pode ler a planilha.
- **Sem testes automatizados** e sem validação do número digitado no formulário (só `required`).
- O rodapé diz "Todos os direitos reservados" sem identificar o titular.

## Histórico

Criado em 06/06/2025 e iterado no mesmo dia: primeiro `index_atualizado.html` e `empreendedor_atualizado.html`, depois renomeados para `index.html` e `empreendedor.html`.

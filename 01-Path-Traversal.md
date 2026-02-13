
# Lab: File path traversal, simple case

## 📝 Descrição do Lab
[cite_start]Este laboratório da **PortSwigger Web Security Academy** [cite: 20] contém uma vulnerabilidade de **Path Traversal** (também conhecida como Directory Traversal) na exibição de imagens de produtos.

## 🎯 Objetivo
[cite_start]Explorar a vulnerabilidade para ler o conteúdo do arquivo `/etc/passwd` no servidor web[cite: 25].

## 🛠️ Ferramentas Utilizadas
* [cite_start]**Burp Suite Professional/Community** (Intercepting Proxy & Repeater) [cite: 28]

## 🛡️ Vulnerabilidade: Path Traversal
[cite_start]A aplicação recebe um parâmetro de nome de arquivo (`filename`) para carregar imagens[cite: 25]. Como o servidor não realiza a **sanitização de inputs** nem valida o caminho do arquivo, é possível navegar pela estrutura de diretórios do sistema operacional.

### Passo a Passo da Exploração
1. [cite_start]**Interceptação:** Utilizei o **Burp Suite** para interceptar a requisição HTTP que carregava uma imagem do produto[cite: 28].
2. **Análise:** Identifiquei o parâmetro `filename=218.png`.
3. **Manipulação:** Enviei a requisição para o **Repeater** e modifiquei o parâmetro para: 
   `../../../../etc/passwd`
4. **Execução:** A sequência `../` instrui o servidor a subir um nível na hierarquia de pastas até atingir a raiz do sistema de arquivos.
5. **Resultado:** O servidor retornou o conteúdo do arquivo `/etc/passwd` no corpo da resposta HTTP.

## 🧠 Lições Aprendidas (Notas Técnicas)
* **Não é Execução de Comando (RCE):** Inicialmente, confundi a falha com execução de comandos, mas aprendi que se trata de uma falha de **leitura de arquivos locais (LFI)** devido à falta de validação de path.
* **Input Sanitization:** A falha ocorre porque o backend confia cegamente no que o usuário envia no parâmetro.
* **Privilégios:** A exploração permite ler arquivos com o nível de permissão do usuário que executa o serviço web (ex: `www-data`), não necessariamente privilégios de root.

## 🚀 Como Mitigar
Para corrigir essa vulnerabilidade, a aplicação deve:
1. Validar o input do usuário contra uma lista de permissões (allowlist).
2. Usar APIs de arquivos que não aceitem caminhos relativos ou absolutos.
3. Garantir que o servidor web rode com privilégios mínimos.

!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Notícias Marinha</title>
  <style>
    
    html, body {
      font-family: Arial, sans-serif;
      background: #e9da08;
      display: flex;
      justify-content: center;
      padding-top: 50px;
    }
    .container {
      background: #0e38e0;
      padding: 20px;
      border-radius: 10px;
      width: 500px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }
    #chat-box {
      height: 300px;
      overflow-y: scroll;
      border: 1px solid #dddddd;
      margin-bottom: 100px;
      padding: 10px;
      background: #fff;
    }
    input[type="text"] {
      width: 70%;
      padding: 10px;
      margin-right: 10px;
    }
    button {
      padding: 10px 15px;
      cursor: pointer;
    }
    .news {
      margin-bottom: 15px;
    }
    .news a {
      color: rgb(16, 14, 141);
      text-decoration: none;
    }
    .news a:hover {
      text-decoration: underline;
    }
    .usuario {
      color: #201f14;
    }
    .Assistente {
      color: #062a52;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Centralizar Notícias</h1>
    <div id="chat-box"></div>
    <input type="text" id="input-text" placeholder="Digite uma palavra-chave..." />
    <button onclick="buscarNoticias()">Buscar</button>
    <button onclick="limparHistorico()">Limpar Histórico</button>
  </div>
  <script>
    const apiKey = "458af66b784f4185bbe0db47341c8d00"; 
    const chatBox = document.getElementById('chat-box');
    const input = document.getElementById('input-text');

    window.onload = carregarHistorico;

    input.addEventListener('keydown', function(e) {
      if (e.key === 'Enter') buscarNoticias();
    });

    function buscarNoticias() {
      const termo = input.value.trim();
      if (!termo) return;

      const mensagemUsuario = `<div class="usuario"><strong>Você:</strong> ${termo}</div>`;
      chatBox.innerHTML += mensagemUsuario;
      salvarNoHistorico(mensagemUsuario);

      input.value = '';

      const domains = 'marinha.mil.br,g1.globo.com,defesaemfoco.com.br';
      const url = `https://newsapi.org/v2/everything?q=${encodeURIComponent(termo)}&domains=${domains}&sortBy=publishedAt&language=pt&pageSize=5&apiKey=${apiKey}`;

      fetch(url)
        .then(res => res.json())
        .then(data => {
          if (data.articles.length === 0) {
            const resposta = `<div class="bot"><strong>Bot:</strong> Nenhuma notícia encontrada para "${termo}".</div>`;
            chatBox.innerHTML += resposta;
            salvarNoHistorico(resposta);
          } else {
            const resposta = `<div class="bot"><strong>Bot:</strong> Resultados encontrados:</div>`;
            chatBox.innerHTML += resposta;
            salvarNoHistorico(resposta);

            data.articles.forEach(article => {
              const dataPublicacao = new Date(article.publishedAt).toLocaleString('pt-BR');
              const noticia = `
                <div class="news">
                  <a href="${article.url}" target="_blank">${article.title}</a><br>
                  <small>${article.source.name} - ${dataPublicacao}</small><br>
                  <p>${article.description || ''}</p>
                </div>
              `;
              chatBox.innerHTML += noticia;
              salvarNoHistorico(noticia);
            });
          }
          chatBox.scrollTop = chatBox.scrollHeight;
        })
        .catch(err => {
          const erro = `<div class="bot"><strong>Bot:</strong> Erro ao buscar notícias.</div>`;
          chatBox.innerHTML += erro;
          salvarNoHistorico(erro);
          console.error(err);
        });
    }

    function salvarNoHistorico(mensagem) {
      const historico = JSON.parse(localStorage.getItem('chat')) || [];
      historico.push(mensagem);
      localStorage.setItem('chat', JSON.stringify(historico));
    }

    function carregarHistorico() {
      const historico = JSON.parse(localStorage.getItem('chat')) || [];
      historico.forEach(msg => chatBox.innerHTML += msg);
      chatBox.scrollTop = chatBox.scrollHeight;
    }

    function limparHistorico() {
      localStorage.removeItem('chat');
      chatBox.innerHTML = '';
    }
  </script>
</body>
</html>

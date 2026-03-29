<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Resumo PDF Positivos</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>

<style>
body { font-family: Arial; background: #111; color: white; text-align: center; padding: 20px; }
h1 { color: #00ffcc; }
input { margin: 20px; padding: 10px; }
table { width: 95%; margin: 20px auto; border-collapse: collapse; }
th, td { border: 1px solid #fff; padding: 8px; text-align: right; }
th { background: #00aa88; color: #fff; }
td:first-child { text-align: left; }
td { background: #063; }
</style>
</head>

<body>

<h1>📄 Resumo de PDFs Positivos</h1>

<input type="file" id="pdfInput" multiple accept="application/pdf">

<table>
  <thead>
    <tr>
      <th>Arquivo</th>
      <th>Produtos (2603)</th>
      <th>Mercadoria (2652)</th>
      <th>Prestação Serviços (2700)</th>
      <th>Simples Nacional (2831)</th>
      <th>Resultado do Período</th>
    </tr>
  </thead>
  <tbody id="tabelaResumo"></tbody>
</table>

<script>
const input = document.getElementById("pdfInput");

input.addEventListener("change", async (event) => {
  document.getElementById("tabelaResumo").innerHTML = "";
  const arquivos = event.target.files;

  for (let file of arquivos) {
    const texto = await lerPDF(file);
    extrairInformacoes(texto, file.name);
  }
});

async function lerPDF(file) {
  const reader = new FileReader();

  return new Promise((resolve) => {
    reader.onload = async function () {
      const typedarray = new Uint8Array(this.result);
      const pdf = await pdfjsLib.getDocument(typedarray).promise;

      let texto = "";
      for (let i = 1; i <= pdf.numPages; i++) {
        const pagina = await pdf.getPage(i);
        const conteudo = await pagina.getTextContent();
        conteudo.items.forEach(item => { texto += item.str + " "; });
        texto += "\n";
      }

      resolve(texto.toLowerCase());
    };

    reader.readAsArrayBuffer(file);
  });
}

function extrairInformacoes(texto, nomeArquivo) {

  texto = texto.replace(/\s+/g, " ");

  // Função para pegar a quarta coluna (saldo) de uma linha
  function pegarSaldoDaLinha(linha) {
    if (!linha) return "-";
    const numeros = linha.match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);
    return numeros && numeros.length >= 4 ? numeros[3].replace(/\s+/g,"") : "-";
  }

  // Função para buscar linha pelo código exato
  function buscarLinha(codigo) {
    const regex = new RegExp(`${codigo}.*?(?=\\d{4}|$)`, "i");
    const match = texto.match(regex);
    return match ? match[0] : "";
  }

  const produtosLinha = buscarLinha("2603");
  const mercadoriaLinha = buscarLinha("2652");
  const servicosLinha = buscarLinha("2700");
  const simplesLinha = buscarLinha("2831");

  const produtos = pegarSaldoDaLinha(produtosLinha);
  const mercadoria = pegarSaldoDaLinha(mercadoriaLinha);
  const servicos = pegarSaldoDaLinha(servicosLinha);
  const simples = pegarSaldoDaLinha(simplesLinha);

  // Resultado do Período
  let resultado = "-";
  const regexResultado = /resultado do período(.{0,200})/i;
  const matchResultado = texto.match(regexResultado);
  if (matchResultado) {
    const trecho = matchResultado[1];
    const numeros = trecho.match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);
    if (numeros && numeros.length >= 1) {
      resultado = numeros[numeros.length - 1].replace(/\s+/g,"");
    }
  }

  const tbody = document.getElementById("tabelaResumo");
  const tr = document.createElement("tr");
  tr.innerHTML = `
    <td>${nomeArquivo}</td>
    <td>${produtos}</td>
    <td>${mercadoria}</td>
    <td>${servicos}</td>
    <td>${simples}</td>
    <td>${resultado}</td>
  `;
  tbody.appendChild(tr);
}
</script>

</body>
</html>

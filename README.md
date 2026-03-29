<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Resumo PDF Positivos</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>

<style>
body {
  font-family: Arial;
  background: #111;
  color: white;
  text-align: center;
  padding: 20px;
}
h1 { color: #00ffcc; }
input { margin: 20px; padding: 10px; }
table {
  width: 90%;
  margin: 20px auto;
  border-collapse: collapse;
}
th, td {
  border: 1px solid #fff;
  padding: 8px;
}
th { background: #00aa88; }
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
      <th>Vendas de Mercadoria</th>
      <th>Prestação de Serviços</th>
      <th>Simples Nacional</th>
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

        conteudo.items.forEach(item => {
          texto += item.str + " ";
        });

        texto += "\n";
      }

      resolve(texto.toLowerCase());
    };

    reader.readAsArrayBuffer(file);
  });
}

function extrairInformacoes(texto, nomeArquivo) {

  texto = texto.replace(/\s+/g, " ");

  // Função auxiliar para pegar o 4º valor da linha
  function pegarSaldo(linha) {
    const valores = linha.match(/\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?/g);
    return valores && valores.length >= 4 ? valores[3].replace(/\s+/g, "") : "-";
  }

  // Procurando linhas específicas pelo código
  const vendasLinha = texto.match(/2652.*?(\d{1,3}(?:\.\d{3})*,\d{2})/i);
  const servicosLinha = texto.match(/2700.*?(\d{1,3}(?:\.\d{3})*,\d{2})/i);
  const simplesLinha = texto.match(/2831.*?(\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?)/i);
  const resultadoLinha = texto.match(/resultado do período.*?(\(?\d{1,3}(?:\.\d{3})*,\d{2}\)?)/i);

  const vendas = vendasLinha ? pegarSaldo(vendasLinha[0]) : "-";
  const servicos = servicosLinha ? pegarSaldo(servicosLinha[0]) : "-";
  const simples = simplesLinha ? pegarSaldo(simplesLinha[0]) : "-";
  const resultado = resultadoLinha ? pegarSaldo(resultadoLinha[0]) : "-";

  const tbody = document.getElementById("tabelaResumo");
  const tr = document.createElement("tr");
  tr.innerHTML = `
    <td>${nomeArquivo}</td>
    <td>${vendas}</td>
    <td>${servicos}</td>
    <td>${simples}</td>
    <td>${resultado}</td>
  `;
  tbody.appendChild(tr);
}
</script>

</body>
</html>

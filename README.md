<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Gerador de Nota de Serviço</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script>
    let servicoId = 1;

    function formatarValor(valor) {
      return valor.replace(/[^\d,]/g, '').replace(',', '.');
    }

    window.adicionarServico = function () {
      const container = document.getElementById('servicos');
      const div = document.createElement('div');
      div.className = 'flex flex-col md:flex-row gap-2 mb-2';
      div.innerHTML = `
        <input type="text" placeholder="Descrição do serviço" class="descricao border p-2 rounded w-full md:w-2/3" required>
        <input type="text" placeholder="Valor (R$)" class="valor border p-2 rounded w-full md:w-1/3" required>
        <button onclick="removerServico(this)" class="text-red-500">Remover</button>
      `;
      container.appendChild(div);
    }

    window.removerServico = function (botao) {
      botao.parentElement.remove();
      atualizarTotal();
    }

    window.atualizarTotal = function () {
      const valores = document.querySelectorAll('.valor');
      let total = 0;
      
      valores.forEach(input => {
        const valor = parseFloat(formatarValor(input.value));
        if (!isNaN(valor)) total += valor;
      });

      document.getElementById('total').textContent = `Total: R$ ${total.toFixed(2)}`;
    }

    window.gerarPDF = async function () {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();

      const nome = document.getElementById('nome').value;
      const data = document.getElementById('data').value;
      const descricoes = document.querySelectorAll('.descricao');
      const valores = document.querySelectorAll('.valor');

      doc.setFontSize(16);
      doc.text('Nota de Serviço', 105, 20, { align: 'center' });

      doc.setFontSize(12);
      doc.text(`Nome: ${nome}`, 10, 35);
      doc.text(`Data: ${data}`, 10, 45);

      doc.text('Serviços:', 10, 60);

      let y = 70;
      let total = 0;

      descricoes.forEach((desc, i) => {
        const texto = `${desc.value} - R$ ${parseFloat(valores[i].value).toFixed(2)}`;
        doc.text(texto, 10, y);
        total += parseFloat(valores[i].value);
        y += 10;
      });

      doc.setFont(undefined, 'bold');
      doc.text(`Total: R$ ${total.toFixed(2)}`, 10, y + 10);

      doc.save(`nota_servico_${nome.replace(/\s+/g, '_')}.pdf`);
    }

    window.gerarCSV = function () {
      const nome = document.getElementById('nome').value;
      const data = document.getElementById('data').value;
      const descricoes = document.querySelectorAll('.descricao');
      const valores = document.querySelectorAll('.valor');

      let csvContent = 'Nome,Data,Descrição do Serviço,Valor (R$)\n';

      let total = 0;

      descricoes.forEach((desc, i) => {
        const descricao = desc.value;
        const valor = parseFloat(formatarValor(valores[i].value)).toFixed(2);
        csvContent += `${nome},${data},${descricao},${valor}\n`;
        total += parseFloat(valores[i].value);
      });

      csvContent += `\nTotal,,${total.toFixed(2)}`;

      const link = document.createElement('a');
      link.href = 'data:text/csv;charset=utf-8,' + encodeURIComponent(csvContent);
      link.target = '_blank';
      link.download = `nota_servico_${nome.replace(/\s+/g, '_')}.csv`;
      link.click();
    }
  </script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body class="min-h-screen bg-gradient-to-br from-gray-100 to-gray-200 py-10 px-4">
  <div class="max-w-3xl mx-auto bg-white p-8 rounded-2xl shadow-2xl">
    <h1 class="text-3xl font-bold text-center mb-6 text-gray-800">Gerador de Nota de Serviço</h1>

    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1">Nome do Cliente</label>
      <input id="nome" type="text" class="border border-gray-300 rounded p-3 w-full" placeholder="Ex: Ana Paula" required>
    </div>

    <div class="mb-6">
      <label class="block text-sm font-medium text-gray-700 mb-1">Data</label>
      <input id="data" type="date" class="border border-gray-300 rounded p-3 w-full" required>
    </div>

    <div class="mb-6">
      <label class="block text-sm font-medium text-gray-700 mb-2">Serviços</label>
      <div id="servicos" class="space-y-3">
        <!-- Serviços serão adicionados aqui -->
      </div>
      <button onclick="adicionarServico()" class="mt-3 bg-blue-600 hover:bg-blue-700 text-white font-medium px-4 py-2 rounded">+ Adicionar Serviço</button>
    </div>

    <div id="total" class="text-center text-xl font-semibold mb-4">Total: R$ 0,00</div>

    <div class="text-center">
      <button onclick="gerarPDF()" class="bg-green-600 hover:bg-green-700 text-white font-semibold text-lg px-6 py-3 rounded-lg shadow-md">Gerar PDF</button>
      <button onclick="gerarCSV()" class="ml-4 bg-yellow-600 hover:bg-yellow-700 text-white font-semibold text-lg px-6 py-3 rounded-lg shadow-md">Gerar CSV</button>
    </div>
  </div>
</body>
</html>

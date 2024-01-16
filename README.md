# chat
teste 1
const SHA256 = require('crypto-js/sha256');

class Transacao {
    constructor(origem, destino, quantidade) {
        this.origem = origem;
        this.destino = destino;
        this.quantidade = quantidade;
    }
}

class Bloco {
    constructor(indice, timestamp, transacoes, hashAnterior = '') {
        this.indice = indice;
        this.timestamp = timestamp;
        this.transacoes = transacoes;
        this.hashAnterior = hashAnterior;
        this.hash = this.calcularHash();
        this.nonce = 0;
    }

    calcularHash() {
        return SHA256(
            this.indice +
            this.timestamp +
            JSON.stringify(this.transacoes) +
            this.hashAnterior +
            this.nonce
        ).toString();
    }

    minerarBloco(dificuldade) {
        while (this.hash.substring(0, dificuldade) !== Array(dificuldade + 1).join('0')) {
            this.nonce++;
            this.hash = this.calcularHash();
        }
        console.log('Bloco minerado: ' + this.hash);
    }
}

class Blockchain {
    constructor() {
        this.chain = [this.criarBlocoGenesis()];
        this.dificuldade = 2;
        this.transacoesPendentes = [];
        this.recompensaMineracao = 100;
    }

    criarBlocoGenesis() {
        return new Bloco(0, '01/01/2022', 'Bloco Gênesis', '0');
    }

    obterUltimoBloco() {
        return this.chain[this.chain.length - 1];
    }

    adicionarTransacao(transacao) {
        this.transacoesPendentes.push(transacao);
    }

    minerarTransacoesPendentes(enderecoMinerador) {
        const recompensa = new Transacao(null, enderecoMinerador, this.recompensaMineracao);
        this.transacoesPendentes.push(recompensa);

        const novoBloco = new Bloco(
            this.obterUltimoBloco().indice + 1,
            new Date().toLocaleString(),
            this.transacoesPendentes,
            this.obterUltimoBloco().hash
        );

        novoBloco.minerarBloco(this.dificuldade);

        console.log('Bloco minerado com sucesso!');
        this.chain.push(novoBloco);

        this.transacoesPendentes = [];
    }

    obterSaldo(endereco) {
        let saldo = 0;
        for (const bloco of this.chain) {
            for (const transacao of bloco.transacoes) {
                if (transacao.origem === endereco) saldo -= transacao.quantidade;
                if (transacao.destino === endereco) saldo += transacao.quantidade;
            }
        }
        return saldo;
    }

    validarBlockchain() {
        for (let i = 1; i < this.chain.length; i++) {
            const blocoAtual = this.chain[i];
            const blocoAnterior = this.chain[i - 1];

            // Verificar se as hashes correspondem
            if (blocoAtual.hash !== blocoAtual.calcularHash()) {
                console.log('A hash do bloco ' + i + ' é inválida.');
                return false;
            }

            // Verificar se a hash anterior corresponde
            if (blocoAtual.hashAnterior !== blocoAnterior.hash) {
                console.log('A hash anterior do bloco ' + i + ' é inválida.');
                return false;
            }
        }

        console.log('A blockchain é válida.');
        return true;
    }
}

// Exemplo de uso:
const minhaBlockchain = new Blockchain();

minhaBlockchain.adicionarTransacao(new Transacao('endereco1', 'endereco2', 10));
minhaBlockchain.adicionarTransacao(new Transacao('endereco2', 'endereco1', 5));

console.log('\nMinerando bloco 1...');
minhaBlockchain.minerarTransacoesPendentes('endereco3');

console.log('\nAdicionando mais transações...');
minhaBlockchain.adicionarTransacao(new Transacao('endereco1', 'endereco2', 2));
minhaBlockchain.adicionarTransacao(new Transacao('endereco2', 'endereco1', 8));

console.log('\nMinerando bloco 2...');
minhaBlockchain.minerarTransacoesPendentes('endereco3');

console.log('\nValidando blockchain...');
minhaBlockchain.validarBlockchain();

console.log('\nSaldo de endereco1:', minhaBlockchain.obterSaldo('endereco1'));
console.log('Saldo de endereco2:', minhaBlockchain.obterSaldo('endereco2'));
console.log('Saldo de endereco3:', minhaBlockchain.obterSaldo('endereco3'));

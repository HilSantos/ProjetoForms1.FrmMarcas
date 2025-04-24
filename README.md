# ProjetoForms1.FrmMarcas
Formulario para cadastro de marcas

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Data.SqlClient;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace ProjetoForms1
{
    public partial class FrmMarcas : Form
    {
        public FrmMarcas()
        {
            InitializeComponent();
            AtualizarGrid();
        }
        //Criar a instância do SQLConnection
        SqlConnection con = new SqlConnection(Dados.Conexao);
        private void btnInserir_Click(object sender, EventArgs e)
        {
            try
            {
                //validar os campos: cpf,nome,tel e cep.
                //se não estão preenchidos-enviar msgbox
                if (txtNome.Text == "") //ou||
                {
                    MessageBox.Show("Preencha o campo obrigatório!",
                        "Sistema TI35", MessageBoxButtons.OK,
                        MessageBoxIcon.Exclamation);
                }
                else
                {
                    //processo de inserir cliente novo
                    //abertura do banco
                    con.Open();
                    //atribuição da instrução do insert na variável sql
                    string sql = "Insert into marca(nome,descricao)values(@nome,@descricao)";
                    //para executar a instrução acima, é necessário executador
                    SqlCommand cmd = new SqlCommand(sql, con);
                    cmd.Parameters.Add("@nome", SqlDbType.VarChar).Value = txtNome.Text;
                    cmd.Parameters.Add("@descricao", SqlDbType.VarChar).Value =txtDescricao.Text;
                    //se todos os campos estiverem ok, vai executar a instrução no banco
                    cmd.ExecuteNonQuery();
                    //fechar conexão
                    con.Close();

  //vai limpar os campos
                    LimparTudo();

  //Atualizar a grid
                    AtualizarGrid();

  //mensagem de confirmação ok
                    MessageBox.Show("Marca ok!", "Sistema Ti35", MessageBoxButtons.OK,
                        MessageBoxIcon.Information);
                }

  }
            catch (SqlException erro)
            {
                MessageBox.Show("Erro ao inserir marca: " + erro.Message);
            }
        }
        private void LimparTudo()
        {
            txtCodigo.Clear();
            txtDescricao.Clear();
            txtNome.Clear();    
            txtNome.Focus();
        }
        private void btnLimpar_Click(object sender, EventArgs e)
        {
            LimparTudo();
        }

  private void btnAlterar_Click(object sender, EventArgs e)
        {
            try
            {
                //declarar uma variável do tipo DialogResult e receber
                //a resposta se deseja alterar
                DialogResult resp = MessageBox.Show("Deseja realmente alterar?",
                    "Sistema TI35", MessageBoxButtons.YesNo,
                    MessageBoxIcon.Question);
                if (resp == DialogResult.Yes)
                {
                    //vai efetuar as alterações daquele cliente em questão
                    MarcaInformation m = new MarcaInformation();
                    m.Codigo = Convert.ToInt32(txtCodigo.Text);
                    m.Nome = txtNome.Text;
                    m.Descricao = txtDescricao.Text;

  //abrir o banco
                    con.Open();
                    //instrução sql para alterar
                    string sqlAlterar = "Update marca set nome=@nome,descricao=@descricao where " +
                        "codigo=@codigo";
                    SqlCommand cmdAlterar = new SqlCommand(sqlAlterar, con);
                    cmdAlterar.Parameters.Add("@nome", SqlDbType.VarChar).Value =
                        m.Nome;
                    cmdAlterar.Parameters.Add("@descricao", SqlDbType.VarChar).Value =
                        m.Descricao;
                    cmdAlterar.Parameters.Add("@codigo", SqlDbType.Int).Value =
                      Convert.ToInt32(m.Codigo);

  //se tudo estiver ok, vai executar a query
                    cmdAlterar.ExecuteNonQuery();
                    con.Close();
                    AtualizarGrid();
                    LimparTudo();
                }
            }
            catch (SqlException erro)
            {
                MessageBox.Show(erro.Message);
            }
        }
        public void AtualizarGrid()
        {
            try
            {
                dgCliente.DataSource = listaMarcas();

  //configuração do cabeçalho do dgCliente
                dgCliente.Columns[0].HeaderText = "Cód.";
                dgCliente.Columns[1].HeaderText = "Nome";
                dgCliente.Columns[2].HeaderText = "Descrição";

  //configuração da largura do dgCliente
                dgCliente.Columns[0].Width = 80;
                dgCliente.Columns[1].Width = 200;
                dgCliente.Columns[2].Width = 250;

  //configuração de permissão
                dgCliente.SelectionMode =
                    DataGridViewSelectionMode.FullRowSelect;
                dgCliente.AllowUserToDeleteRows = false;
                dgCliente.AllowUserToAddRows = false;
                dgCliente.ReadOnly = true;
            }
            catch (SqlException erro)
            {
                MessageBox.Show("Erro: " + erro.Message);
            }
        }
   
  public static DataTable listaMarcas()
        {
            try
            {
                SqlConnection con = new SqlConnection(Dados.Conexao);
                con.Open();
                string sqlListar = "Select * from marca";
                //vamos utilizar uma classe adaptador para receber os
                //dados da tabela cliente
                SqlDataAdapter da = new SqlDataAdapter(sqlListar, con);
                //estamos chamando uma classe do tipo Tabela
                DataTable dt = new DataTable();
                da.Fill(dt);
                return dt;
                con.Close();
            }
            catch (SqlException erro)
            {
                return null;
            }
        }

  private void dgCliente_MouseClick(object sender, MouseEventArgs e)
        {
            try
            {
                //vai capturar qual é a linha selecionada e qual é
                //o código do cliente em questão

  //identificar qual é a linha clicada
                int linha = dgCliente.SelectedRows[0].Index;
                //MessageBox.Show(""+linha);

  //analisar a linha clicada
                if (linha >= 0)
                {
                    //precisa saber qual é o código do cliente ref. 
                    //a linha clicada
                    int codigo = Convert.ToInt32
                        (dgCliente.Rows[linha].Cells[0].Value);
                    //MessageBox.Show("Linha: " + linha + " e Código: " + codigo);
                    //acesso ao método selecionarCliente,
                    //onde fará a busca pelo código
                    MarcaInformation m = selecionarMarca(codigo);
                    txtCodigo.Text = m.Codigo.ToString();
                    txtNome.Text = m.Nome.ToString();
                    txtDescricao.Text = m.Descricao.ToString();
                }
            }
            catch (SqlException erro)
            {

  }
        }
        public static MarcaInformation selecionarMarca(int codigo)
        {
            try
            {
                SqlConnection con = new SqlConnection(Dados.Conexao);
                con.Open();
                string sqlSelecionar = "Select * from marca where codigo=@codigo";
                SqlCommand cmd = new SqlCommand(sqlSelecionar, con);
                cmd.Parameters.Add("@codigo", SqlDbType.Int).Value = codigo;
                //como precisa trazer apenas as inf´s do código específico, ele vai
                //varrer os registros e trazer somente o que for válido
                SqlDataReader dr = cmd.ExecuteReader();
                //se houver dados para efetuar a busca, vai varrer até o último reg.
                if (dr.Read())
                {
                    MarcaInformation m = new MarcaInformation();
                    m.Codigo = Convert.ToInt32(dr[0]);
                    m.Nome = dr[1].ToString();
                    m.Descricao = dr[2].ToString();
                    
  con.Close();
                    return m;
                }
                else
                {
                    con.Close();
                    return null;
                }

  }
            catch (SqlException erro)
            {
                return null;
            }
        }

  private void txtPesquisaNome_TextChanged(object sender, EventArgs e)
        {
            try
            {
                dgCliente.DataSource = pesquisaMarcas(txtPesquisaNome.Text);

  //configuração do cabeçalho do dgCliente
                dgCliente.Columns[0].HeaderText = "Cód.";
                dgCliente.Columns[1].HeaderText = "Nome";
                dgCliente.Columns[2].HeaderText = "Descrição";

  //configuração da largura do dgCliente
                dgCliente.Columns[0].Width = 80;
                dgCliente.Columns[1].Width = 200;
                dgCliente.Columns[2].Width = 250;

  //configuração de permissão
                dgCliente.SelectionMode =
                    DataGridViewSelectionMode.FullRowSelect;
                dgCliente.AllowUserToDeleteRows = false;
                dgCliente.AllowUserToAddRows = false;
                dgCliente.ReadOnly = true;
            }
            catch (SqlException erro)
            {
                MessageBox.Show("Erro: " + erro.Message);
            }
        }
        public static DataTable pesquisaMarcas(string nome)
        {
            try
            {
                SqlConnection con = new SqlConnection(Dados.Conexao);
                con.Open();
                string sqlPesquisar =
                    "Select * from marca where nome like @nome";
                SqlDataAdapter da = new SqlDataAdapter(sqlPesquisar, con);
                da.SelectCommand.Parameters.Add("@nome", SqlDbType.VarChar).Value =
                    "%" + nome + "%";
                DataTable dt = new DataTable();
                da.Fill(dt);
                return dt;
                con.Close();
            }
            catch (SqlException erro)
            {
                MessageBox.Show("Erro ao Pesquisar o nome da Marca!"
                    + erro.Message);
                return null;
            }
        }

 private void FrmMarcas_Load(object sender, EventArgs e)
        {

  }
}
}

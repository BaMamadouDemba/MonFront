<template>
  <div>
    <header class="header">
      <h1>Finao</h1>
    </header>
    <main class="main-content">
      <!-- Formulaire pour envoyer le fichier -->
      <form @submit.prevent="submitFile" class="file-form">
        <input type="file" ref="fileInput" />
        <button type="submit" class="submit-button">Envoyer le fichier</button>
      </form>
      <p v-if="message" class="message">{{ message }}</p>

      <!-- Bouton pour importer les données -->
      <button @click="loadData" class="import-button">Importer les données</button>

       <div>
        <button @click="exportXml">Exporter en XML</button>
    </div>

      <!-- Tableau des données importées -->
      <h2>Données Importées</h2>
      <table v-if="records.length > 0" class="data-table">
        <thead>
          <tr>
            <!-- Affichage dynamique des colonnes -->
            <th v-for="(column, index) in columnNames" :key="index">{{ column }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(record, index) in records" :key="index">
            <!-- Affichage dynamique des valeurs des colonnes -->
            <td v-for="(value, colIndex) in record" :key="colIndex">{{ value }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>Aucune donnée à afficher.</p>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      records: [], // Contiendra les données importées sous forme d'un tableau
      columnNames: [], // Contiendra les noms des colonnes
      message: '', // Message pour l'état d'importation
    };
  },
  methods: {


 exportXml() {
            // Appeler la route backend pour télécharger le fichier
            window.location.href = "http://localhost:8082/export-xml";
        },

    // Méthode pour soumettre le fichier sans charger les données automatiquement
    submitFile() {
      const fileInput = this.$refs.fileInput;
      const formData = new FormData();
      formData.append('file', fileInput.files[0]);

      fetch('http://localhost:8082/api/excel/import', {
        method: 'POST',
        body: formData,
      })
        .then((response) => {
          if (!response.ok) {
            throw new Error("Erreur lors de l'importation du fichier");
          }
          this.message = 'Fichier importé avec succès !';
          // Pas de chargement des données ici
        })
        .catch((error) => {
          console.error("Erreur lors de l'importation:", error);
          this.message = "Erreur lors de l'importation.";
        });
    },

    // Méthode pour charger les données dynamiquement
    async loadData() {
      try {
        const response = await fetch('http://localhost:8082/api/excel/data');
        if (!response.ok) {
          throw new Error('Erreur de réseau');
        }
        const fetchedData = await response.json();
        console.log('Données récupérées :', fetchedData);

        // Traitement des données reçues
        if (fetchedData.length > 0) {
          // Récupérer les colonnes uniques à partir des premiers éléments
          this.columnNames = [...new Set(fetchedData.flatMap(item => item.columns.map(col => col.column_name)))];

          // Créer les enregistrements en mappant les valeurs des colonnes
          this.records = fetchedData.map(item => {
            const record = {};
            this.columnNames.forEach(colName => {
              const column = item.columns.find(col => col.column_name === colName);
              record[colName] = column ? column.column_value : '';
            });
            return record;
          });
        }
      } catch (error) {
        console.error('Erreur lors du chargement des données:', error);
        this.records = [];
      }
    },
  },
};
</script>

<style scoped>
  .data-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }

  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }

  th {
    background-color: red;
  }
</style>
<script lang="ts">
  //@ts-ignore
  import Papa from "papaparse";
  //@ts-ignore
  import proj4 from "proj4";

  /** The csv file list */
  let file: FileList;
  /** The new file name */
  let newName: string;
  /** Latitude column name */
  let latColName: string;
  /** Longitude column name */
  let longColName: string;
  /** Columns name to keep */
  let colsToKeep: string | string[] = "";
  /** Columns name to join into one */
  let colsToJoin: string | string[] = "";
  /** The end data */
  let data: any = [];

  /** CSV content */
  let csvContent = "data:text/csv;charset=utf-8";

  const openFile = () => {
    const csv = Papa.unparse(data);
    console.log(csv);

    const csvData = new Blob([csv], { type: "text/csv;charset=utf-8;" });
    let csvUrl = null;

    //@ts-ignore
    if (navigator.msSaveBlob) {
      csvURL = navigator.msSaveBlob(csvData, "download.csv");
    } else {
      csvUrl = window.URL.createObjectURL(csvData);
    }

    console.log(csvUrl);
    let tempLink = document.createElement("a");
    tempLink.href = csvUrl!;
    tempLink.setAttribute("download", newName + ".csv");
    tempLink.click();
  };

  /**
   * Converting the chosen files
   */
  const getData = () => {
    if (file && file.length === 1) {
      Papa.parse(file[0], {
        header: true,
        complete: (results: any) => {
          convertData(results.data);
        },
      });
    }
  };

  /**
   * Converting the values of the parsed file
   */
  const convertData = (results: any) => {
    let newRow: string[] = [];

    if (colsToKeep.length > 0) {
      //@ts-ignore
      colsToKeep = colsToKeep.split(",");
    }
    if (colsToJoin.length > 0) {
      //@ts-ignore
      colsToJoin = colsToJoin.split(",");
    }

    results.forEach((row: any) => {
      //getting lat and long
      const lat = row[latColName];
      const long = row[longColName];

      if (lat && long) {
        //setting source => destination
        //@ts-ignore
        proj4.defs(
          "EPSG:27572",
          "+proj=lcc +lat_1=46.8 +lat_0=46.8 +lon_0=0 +k_0=0.99987742 +x_0=600000 +y_0=2200000 +a=6378249.2 +b=6356515 +towgs84=-168,-60,320,0,0,0,0 +pm=paris +units=m +no_defs"
        );
        //@ts-ignore
        var source = new proj4.Proj("EPSG:27572"); //Lambert II
        //@ts-ignore
        var destination = new proj4.Proj("WGS84");
        console.log(lat, long);
        //converting data
        const latLong = proj4.transform(source, destination, [
          parseInt(long),
          parseInt(lat),
        ]);

        const mapLink = `https://www.google.com/maps/place/${latLong.y},${latLong.x}`;

        //pushing link and coordinates
        newRow.push(mapLink, lat, long, latLong.x, latLong.y);

        //setting columns to keep
        if (colsToKeep.length > 0) {
          //@ts-ignore
          colsToKeep.forEach((col: string) => {
            newRow.push(row[col.trim()]);
          });
        }

        if (colsToJoin.length > 0) {
          let result = "";
          //@ts-ignore
          colsToJoin.forEach((col: string) => {
            result += row[col.trim()] + "\n";
          });

          console.log(result);
          newRow.push(result);
        }

        data.push(newRow);

        newRow = [];
      }
    });

    //adding new row with columns name
    let colName = [
      "Lien maps",
      latColName,
      longColName,
      "latitude (decimal)",
      "longitude (decimal)",
    ];

    if (colsToKeep.length > 0) {
      colName.push(...colsToKeep);
    }

    if (colsToJoin.length > 0) {
      colName.push("Colonnes jointes");
    }

    data = [colName, ...data];

    console.log(data);
  };
</script>

<form>
  <input required type="file" accept=".csv" bind:files={file} />
  <div class="label-input">
    <label for="lat">Colonne latitude (Y)</label>
    <input required type="text" name="lat" id="lat" bind:value={latColName} />
  </div>
  <div class="label-input">
    <label for="long">Colonne longitude (X)</label>
    <input
      required
      type="text"
      name="long"
      id="long"
      bind:value={longColName}
    />
  </div>
  <div class="label-input">
    <label for="keep">Colonnes à conserver (séparer par des virgules)</label>
    <textarea required name="keep" id="keep" bind:value={colsToKeep} />
  </div>
  <div class="label-input">
    <label for="colsToJoin">Colonnes à joindre</label>
    <textarea
      required
      name="colsToJoin"
      id="colsToJoin"
      bind:value={colsToJoin}
    />
  </div>
  <div class="label-input">
    <label for="newName">Nom du fichier converti</label>
    <input
      required
      type="text"
      name="newName"
      id="newName"
      bind:value={newName}
    />
  </div>
  <div class="label-input">
    <button on:click={getData}>Convertir</button>
    <button on:click={openFile}>Ouvrir</button>
  </div>
</form>

<style>
  form {
    display: flex;
    flex-flow: column nowrap;
    justify-content: center;
    align-items: center;
    gap: 25px;
  }
  .label-input {
    display: flex;
    flex-flow: column nowrap;
    justify-content: center;
    align-items: center;
    gap: 5px;
  }
</style>

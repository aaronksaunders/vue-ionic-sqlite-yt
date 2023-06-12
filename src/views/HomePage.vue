<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Vue + SQLITE</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <template v-if="editItem">
        <ion-item>
          <ion-input type="text" v-model="inputName"></ion-input>
          <ion-button @click="() => setEditItem(undefined)">CANCEL</ion-button>
          <ion-button @click="updateItem">UPDATE</ion-button>
        </ion-item>
      </template>
      <template v-else>
        <ion-item>
          <ion-input type="text" v-model="inputName"></ion-input>
          <ion-button slot="end" @click="addItem">ADD</ion-button>
        </ion-item>
      </template>
      <h3>THE DATA</h3>
      <ion-item v-for="item in items" :key="item?.id">
        <ion-label>
          {{ item.name }}
        </ion-label>
        <ion-button @click="() => setEditItem(item)">EDIT</ion-button>
        <ion-button @click="() => deleteItem(item.id)">DELETE</ion-button>
      </ion-item>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
type SQLItem = {
  id: number;
  name: string;
}
import { ref } from "vue";
import {
  IonContent,
  onIonViewDidEnter,
  onIonViewWillLeave,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonItem,
  IonLabel,
  IonButton,
  IonInput,
} from "@ionic/vue";
import {
  CapacitorSQLite,
  SQLiteConnection,
  SQLiteDBConnection,
} from "@capacitor-community/sqlite";
const items = ref<SQLItem[] | undefined>();
const db = ref<SQLiteDBConnection>();
const sqlite = ref<SQLiteConnection>();
const inputName = ref<string>("");

const editItem = ref<SQLItem | undefined>(undefined);

onIonViewDidEnter(async () => {
  sqlite.value = new SQLiteConnection(CapacitorSQLite);
  const ret = await sqlite.value.checkConnectionsConsistency();
  const isConn = (await sqlite.value.isConnection("db_vite", false)).result;

  if (ret.result && isConn) {
    db.value = await sqlite.value.retrieveConnection("db_vite", false);
  } else {
    db.value = await sqlite.value.createConnection(
      "db_vite",
      false,
      "no-encryption",
      1,
      false
    );
  }

  loadData();
});

onIonViewWillLeave(async () => {
  await sqlite.value?.closeConnection("db_vite", false);
});

/**
 * do an insert to database
 */
const deleteItem = async (id: number) => {
  try {
    await db.value?.open();
    // add test record to db
    await db.value?.query(`DELETE FROM test WHERE id=?;`, [id]);

    // update ui
    const respSelect = await db.value?.query(`SELECT * FROM test`);
    items.value = respSelect?.values;
  } catch (error) {
    alert((error as Error).message);
  } finally {
    inputName.value = "";
    await db.value?.close();
  }
};

/**
 * do an insert to database
 */
const addItem = async () => {
  try {
    await db.value?.open();
    // add test record to db
    await db.value?.query(`INSERT INTO test (id,name) values (?,?);`, [
      Date.now(),
      inputName.value,
    ]);

    // update ui
    const respSelect = await db.value?.query(`SELECT * FROM test;`);
    items.value = respSelect?.values;
  } catch (error) {
    alert((error as Error).message);
  } finally {
    inputName.value = "";
    await db.value?.close();
  }
};

/**
 *
 * @param item
 */
const setEditItem = (item: SQLItem | undefined) => {
  if (item) {
    editItem.value = item;
    inputName.value = item.name;
  } else {
    editItem.value = undefined;
    inputName.value = "";
  }
};

/**
 * update entry in database
 */
const updateItem = async () => {
  try {
    debugger;
    await db.value?.open();
    // update test record to db
    await db.value?.query(`UPDATE test SET name=? WHERE id=?`, [
      inputName.value,
      editItem.value?.id,
    ]);

    setEditItem(undefined);

    // update ui
    const respSelect = await db.value?.query(`SELECT * FROM test;`);
    items.value = respSelect?.values;
  } catch (error) {
    alert((error as Error).message);
  } finally {
    inputName.value = "";
    await db.value?.close();
  }
};

/**
 * do a select of the database
 */
const loadData = async () => {
  try {
    // query db
    await db.value?.open();
    const respSelect = await db.value?.query(`SELECT * FROM test`);
    items.value = respSelect?.values;
  } catch (error) {
    alert((error as Error).message);
  } finally {
    await db.value?.close();
  }
};
</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>

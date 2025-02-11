<!-- <script lang="ts" setup>
import { sql } from "kysely";
import Database, { sqlite } from "./database";
import Migrator from "./database/BrowserMigrator";
import { onMounted } from "vue";

// see: plugins/sqlite.ts for registering jeep-sqlite on the web
// see: database/BrowserMigrator to have a look on implementing migraitons on the web
onMounted(async () => {
  await Migrator.migrate();

  // query builder
  console.log(
    await Database.insertInto("test").values({ name: "wecler" }).execute()
  );
  console.log(await Database.selectFrom("test").selectAll().execute());

  // raw queries
  console.log(
    await sql`INSERT INTO test VALUES (null, 'Dawid') RETURNING *`.execute(
      Database
    )
  );

  // you can also use transactions
  await Database.transaction().execute(async (trx) => {
    await trx
      .insertInto("test")
      .values({ name: "Sami" })
      .returning("id")
      .executeTakeFirstOrThrow();

    return await trx
      .insertInto("test")
      .values({ name: "commit" })
      .returningAll()
      .execute();
  });

  console.log(await Database.selectFrom("test").selectAll().execute());

  try {
    // and rollback
    await Database.transaction().execute(async (trx) => {
      await trx.insertInto("test").values({ name: "Rollback" }).execute();
      await trx.insertInto("test").values({ name: "Rollback" }).execute();
      throw 1;
    });
  } catch (e) {
    //
  }

  console.log(await Database.selectFrom("test").selectAll().execute());

  // this also works but...
  // if the migrator is user you have to ensure that migrator runs before all the queries
  // will be executed therefore it's probably better to use sql from kysely
  const db = await sqlite.retrieveConnection("nuxt", false);

  await db.beginTransaction();
  await db.query("INSERT INTO `test` VALUES (null, ?)", ["Sami"]);
  await db.query("INSERT INTO `test` VALUES (null, ?)", ["commit"]);
  await db.commitTransaction();

  console.log(await db.query("SELECT * FROM test"));

  await db.beginTransaction();
  await db.query("INSERT INTO `test` VALUES (null, ?)", ["Rollback"]);
  await db.query("INSERT INTO `test` VALUES (null, ?)", ["Rollback"]);
  await db.rollbackTransaction();

  await db.execute("INSERT INTO `test` VALUES (null, 'Jean')");

  console.log(await db.query("SELECT `id`, `name` FROM test"));
});
</script>

<template>
  <div>
    <NuxtWelcome />
  </div>
</template> -->

<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { sql } from "kysely";
import Database, { sqlite } from "./database";
import Migrator from "./database/BrowserMigrator";
import {
  IonPage,
  IonContent,
  IonLabel,
  IonItem,
  IonList,
  IonTitle,
  IonToolbar,
  IonHeader,
  IonInput,
  IonButton,
  // IonLabel,
  // IonItem,
  // IonList,
  // IonTitle
} from "@ionic/vue";

const note = ref("");
const notes = ref<{ id: number; content: string }[]>([]);

onMounted(async () => {
  await Migrator.migrate();

  loadNotes();

  // Example transactions and queries
  // console.log(
  //   await Database.insertInto("test").values({ name: "wecler" }).execute()
  // );
  // console.log(await Database.selectFrom("test").selectAll().execute());

  // console.log(
  //   await sql`INSERT INTO test VALUES (null, 'Dawid') RETURNING *`.execute(
  //     Database
  //   )
  // );

  // await Database.transaction().execute(async (trx) => {
  //   await trx
  //     .insertInto("test")
  //     .values({ name: "Sami" })
  //     .returning("id")
  //     .executeTakeFirstOrThrow();
  //   return await trx
  //     .insertInto("test")
  //     .values({ name: "commit" })
  //     .returningAll()
  //     .execute();
  // });

  // console.log(await Database.selectFrom("test").selectAll().execute());

  // try {
  //   await Database.transaction().execute(async (trx) => {
  //     await trx.insertInto("test").values({ name: "Rollback" }).execute();
  //     await trx.insertInto("test").values({ name: "Rollback" }).execute();
  //     throw 1;
  //   });
  // } catch (e) {}

  // console.log(await Database.selectFrom("test").selectAll().execute());

  const db = await sqlite.retrieveConnection("nuxt", false);

  // await db.beginTransaction();
  // await db.query("INSERT INTO `test` VALUES (null, ?)", ["Sami"]);
  // await db.query("INSERT INTO `test` VALUES (null, ?)", ["commit"]);
  // await db.commitTransaction();

  // console.log(await db.query("SELECT * FROM test"));

  // await db.beginTransaction();
  // await db.query("INSERT INTO `test` VALUES (null, ?)", ["Rollback"]);
  // await db.query("INSERT INTO `test` VALUES (null, ?)", ["Rollback"]);
  // await db.rollbackTransaction();

  //   await db.execute("INSERT INTO `test` VALUES (null, 'Jean')");
  //   console.log(await db.query("SELECT `id`, `name` FROM test"));
});

const loadNotes = async () => {
  notes.value = await Database.selectFrom("notes").selectAll().execute();
};

const addNote = async () => {
  if (!note.value.trim()) return;
  await Database.insertInto("notes").values({ content: note.value }).execute();
  note.value = "";
  loadNotes();
};
</script>

<!-- <template>
  <div class="container p-4">
    <h1 class="text-xl font-bold mb-4">Notebook</h1>
    <input
      v-model="note"
      type="text"
      placeholder="Write a note..."
      class="border p-2 w-full mb-2"
    />
    <button @click="addNote" class="bg-blue-500 text-white px-4 py-2 rounded">
      Save Note
    </button>

    <div class="mt-4">
      <h2 class="text-lg font-semibold">Saved Notes:</h2>
      <ul>
        <li v-for="n in notes" :key="n.id" class="border p-2 mt-2">
          {{ n.content }}
        </li>
      </ul>
    </div>
  </div>
</template> -->
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Notebook</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-input
        v-model="note"
        placeholder="Write a note..."
        class="ion-margin-bottom"
      ></ion-input>

      <ion-button expand="full" @click="addNote" color="primary">
        Save Note
      </ion-button>

      <ion-list class="ion-margin-top">
        <ion-item v-for="n in notes" :key="n.id">
          <ion-label>{{ n.content }}</ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

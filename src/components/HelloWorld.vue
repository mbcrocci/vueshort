<template>
  <div>
  <v-card class="elevation-12">
    <v-toolbar color="primary" dark flat>
      <v-spacer />
      <v-toolbar-title>VueShort</v-toolbar-title>
      <v-spacer />
    </v-toolbar>
      <v-card-text>
        <v-text-field
          label="Url"
          placeholder="http://google.com"
          outlined
          v-model="url"
          v-on:keyup.enter="saveOrGoto"
        />
    </v-card-text>
    <v-card-actions>
      <v-spacer />
      <v-btn color="primary" v-on:click="save">Save</v-btn>
      <v-btn color="primary" v-on:click="goto">Go</v-btn>
    </v-card-actions>
    <v-divider />
    <v-list>
      <template v-for="url in this.urls$">
        <v-list-item :key="url.shortid">
          <v-list-item-title v-html="url.shortid"></v-list-item-title>
          <v-list-item-content v-html="url.longUrl"></v-list-item-content>
          {{url.longUrl}}
        </v-list-item>
      </template>
    </v-list>
  </v-card>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { timer } from 'rxjs';
import { switchMap, concatMap, catchError } from 'rxjs/operators';
import { fromFetch } from 'rxjs/fetch';

function isUrl(url: string) {
  const urlRegex = /[-a-zA-Z0-9@:%._+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_+.~#?&//=]*)/g;

  return urlRegex.test(url);
}

function prepareUrl(url: string) {
  let u = url;
  if (!url.includes('http://')) {
    u = `http:///${url}`;
  }

  return u;
}

const apiUrl = 'http://localhost:3000/urls';

export default Vue.extend({
  data() {
    return {
      url: '',
    };
  },
  name: 'HelloWorld',
  subscriptions() {
    const urls$ = timer(0, 1000)
      .pipe(concatMap(() => fromFetch(`${apiUrl}?filter[limit]=10&filter[order]=date DESC`)))
      .pipe(switchMap((response: Response) => response.json()));

    return {
      urls$,
    };
  },
  methods: {
    save() {
      fetch(apiUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          longUrl: this.url,
        }),
      })
        .catch((reason: any) => console.log(reason));
    },
    goto() {
      fetch(`${apiUrl}/${this.url}`)
        .then((resp: Response) => resp.json())
        .then((data: any) => {
          const url = prepareUrl(data.longUrl);
          window.location.href = url;
        })
        .catch((reason: any) => console.log(reason));
    },
    saveOrGoto() {
      if (isUrl(this.url)) {
        this.save();
      } else {
        this.goto();
      }
    },
  },
});
</script>

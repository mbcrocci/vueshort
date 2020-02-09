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
        <v-chip v-if="!this.isCustom"
                class="m-2"
                x-small
                outlined
                v-on:click="setIsCustom">
          Custom Tag
        </v-chip>
        <v-text-field
          label="Custom ID"
          placeholder="Myawesomesite"
          outlined
          v-model="customTag"
          v-else
          v-on:keyup.enter="save"
          />
    </v-card-text>
    <v-card-actions>
      <v-spacer />
      <v-btn color="primary" @click="save">Save</v-btn>
      <v-btn color="primary" @click="goto">Go</v-btn>
    </v-card-actions>
    <v-list>
      <template v-for="url in this.urls$">
        <v-divider :key="'spacer' + url.shortid"/>
        <v-list-item :key="url.shortid"
                      @click="gotoUrl(url)">
          <v-list-item-content>
            <v-list-item-title>{{url.shortid}}</v-list-item-title>
            <v-list-item-subtitle>{{url.longUrl}}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </template>
    </v-list>
  </v-card>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { timer } from 'rxjs';
import { switchMap, concatMap } from 'rxjs/operators';
import { fromFetch } from 'rxjs/fetch';

function isUrl(url: string) {
  const urlRegex = /[-a-zA-Z0-9@:%._+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_+.~#?&//=]*)/g;

  return urlRegex.test(url);
}

function prepareUrl(url: string) {
  let u = url;
  if (!url.includes('http://') || !url.includes('https://')) {
    u = `http:///${url}`;
  }

  return u;
}

const apiUrl = 'https://powerful-mesa-76061.herokuapp.com/urls';

export default Vue.extend({
  data() {
    return {
      url: '',
      customTag: '',
      isCustom: false,
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
      const urlToSave = {
        longUrl: this.url,

        ...this.isCustom && { shortid: this.customTag },
      };

      fetch(apiUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(urlToSave),
      })
        .then(() => { this.isCustom = false; })
        .catch((reason: any) => reason);
    },
    goto() {
      fetch(`${apiUrl}/${this.url}`)
        .then((resp: Response) => resp.json())
        .then((url: any) => this.gotoUrl(url))
        .catch((reason: any) => reason);
    },
    saveOrGoto() {
      if (isUrl(this.url)) {
        this.save();
      } else {
        this.goto();
      }
    },
    gotoUrl(url: any) {
      window.location.href = prepareUrl(url.longUrl);
    },
    setIsCustom() {
      this.isCustom = true;
    },
  },
});
</script>

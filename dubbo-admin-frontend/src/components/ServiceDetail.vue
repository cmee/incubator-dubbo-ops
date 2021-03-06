<!--
  - Licensed to the Apache Software Foundation (ASF) under one or more
  - contributor license agreements.  See the NOTICE file distributed with
  - this work for additional information regarding copyright ownership.
  - The ASF licenses this file to You under the Apache License, Version 2.0
  - (the "License"); you may not use this file except in compliance with
  - the License.  You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <v-container grid-list-xl fluid >
    <v-layout row wrap >
      <v-flex sm12>
        <h3>{{$t('basicInfo')}}</h3>
      </v-flex>
      <v-flex lg12>
        <v-data-table
          :items="basic"
          class="elevation-1"
          hide-actions
          hide-headers >
          <template slot="items" slot-scope="props">
            <td>{{props.item.name}} </td>
            <td>{{props.item.value}}</td>
          </template>
        </v-data-table>
      </v-flex>
      <v-flex sm12>
        <h3>{{$t('serviceInfo')}}</h3>
      </v-flex>
      <v-flex lg12 >
        <v-tabs
          class="elevation-1">
          <v-tab>
            {{$t('providers')}}
          </v-tab>
          <v-tab>
            {{$t('consumers')}}
          </v-tab>
          <v-tab-item>
            <v-data-table
              class="elevation-1"
              :headers="detailHeaders.providers"
              :items="providerDetails"
            >
              <template slot="items" slot-scope="props">
                <td>{{getIp(props.item.address)}}</td>
                <td>{{getPort(props.item.address)}}</td>
                <td></td>
                <td></td>
                <td><v-tooltip top>
                  <v-btn
                          small
                          slot="activator"
                          color="primary"
                  >
                    URL
                  </v-btn>
                  <span>{{props.item.url}}</span>
                </v-tooltip></td>
              </template>
            </v-data-table>
          </v-tab-item>
          <v-tab-item >
            <v-data-table
              class="elevation-1"
              :headers="detailHeaders.consumers"
              :items="consumerDetails"
            >
              <template slot="items" slot-scope="props">
                <td>{{getIp(props.item.address)}}</td>
                <td>{{getPort(props.item.address)}}</td>
                <td>{{props.item.application}}</td>
              </template>
            </v-data-table>
          </v-tab-item>
        </v-tabs>
      </v-flex>
      <v-flex sm12>
        <h3>{{$t('metaData')}}</h3>
      </v-flex>
      <v-flex lg12>
        <v-data-table
          class="elevation-1"
          :headers="metaHeaders"
          :items="methodMetaData">
          <template slot="items" slot-scope="props">
            <td>{{props.item.name}}</td>
            <td>
              <v-chip v-for="(type, index) in props.item.parameterTypes" :key="type.id" label>{{type}}</v-chip>
            </td>
            <td>
              <v-chip label>{{props.item.returnType}}</v-chip>
            </td>
          </template>
          <template slot="no-data">
            <v-alert :value="true" color="warning" icon="warning">
              {{$t('noMetadataHint')}}
              <a :href="$t('configAddress')" target="_blank">{{$t('here')}}</a>
            </v-alert>
          </template>
        </v-data-table>
      </v-flex>
    </v-layout>
  </v-container>
</template>
<script>
  export default {
    data: () => ({
      metaHeaders: [],
      detailHeaders: {},
      providerDetails: [],
      consumerDetails: [],
      methodMetaData: [],
      basic: []
    }),
    methods: {
      setmetaHeaders: function () {
        this.metaHeaders = [
          {
            text: this.$t('methodName'),
            value: 'method',
            sortable: false
          },
          {
            text: this.$t('parameterList'),
            value: 'parameter',
            sortable: false
          },
          {
            text: this.$t('returnType'),
            value: 'returnType',
            sortable: false
          }
        ]
      },
      setdetailHeaders: function () {
        this.detailHeaders = {
          providers: [
            {
              text: this.$t('ip'),
              value: 'ip'
            },
            {
              text: this.$t('port'),
              value: 'port'
            },
            {
              text: this.$t('timeout'),
              value: 'timeout'
            },
            {
              text: this.$t('serialization'),
              value: 'serial'
            },
            {
              text: this.$t('operation'),
              value: 'operate'
            }

          ],
          consumers: [
            {
              text: this.$t('ip'),
              value: 'ip'
            },
            {
              text: this.$t('port'),
              value: 'port'
            },
            {
              text: this.$t('appName'),
              value: 'appName'
            }
          ]
        }
      },
      detail: function (service) {
        this.$axios.get('/service/' + service)
            .then(response => {
              this.providerDetails = response.data.providers
              this.consumerDetails = response.data.consumers
              if (response.data.metadata !== null) {
                this.methodMetaData = response.data.metadata.methods
              }
            })
      },
      getIp: function (address) {
        return address.split(':')[0]
      },
      getPort: function (address) {
        return address.split(':')[1]
      },
      getParameters: function (parameterTypes) {
        let result = ''
        for (let i = 0; i < parameterTypes.length; i++) {
          result = result + parameterTypes[i] + ' '
        }
        return result.trim()
      }
    },
    mounted: function () {
      this.setmetaHeaders()
      this.setdetailHeaders()
      let query = this.$route.query
      let meta = {
        'service': '',
        'app': '',
        'group': '',
        'version': ''
      }
      var vm = this
      Object.keys(query).forEach(function (key) {
        if (key in meta) {
          meta[key] = query[key]
        }
      })
      let dataId = meta['service']
      if (meta['group'] !== '') {
        dataId = meta['group'] + '*' + dataId
      }
      if (meta['version'] !== '') {
        dataId = dataId + ':' + meta['version']
      }

      if (dataId !== '') {
        this.detail(dataId)
        Object.keys(meta).forEach(function (key) {
          let item = {}
          item.value = meta[key]
          item.name = vm.$t(key)
          vm.basic.push(item)
        })
      }
    }
  }
</script>


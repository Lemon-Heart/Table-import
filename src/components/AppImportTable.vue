<template>
  <div class="table">
    <div class="table__methods">
      <app-input-file class="table__import" @onChange="toImport" />
      <button v-if="tableHead" class="table__export" @click="toExport">
        <span>Export File</span>
        <img :src="exportIcon" alt="">
      </button>
      <button v-if="tableHead" class="table__add" :disabled="searchInput ? true : false" @click="addRow">Add row</button>
      <input v-if="tableHead" v-model.trim="editInput" ref="toEditCell" class="table__edit" type="text" @click.stop @focusout="change" @keydown.enter="change">
      <input v-if="tableHead" v-model.trim="searchInput" placeholder="Search..." class="table__search" type="text">
    </div>
    <table v-if="tableHead" class="table__body" id="table">
      <thead>
        <tr v-if="tableHead" class="table__row table__row--head" :style="`grid-template-columns: repeat(${tableHead.length}, 1fr)`">
          <th class="table__cell" v-for="(th, i) in tableHead" :key="i" @click="editHeadCell(i, $event)">
            {{ th }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr class="table__row" v-for="(tr, j) in found" :key="j" :style="`grid-template-columns: repeat(${tableHead.length}, 1fr)`">
          <div class="table__rowMethods" @mouseenter="showMethods" @mouseleave="hideMethods">
            <div v-show="j !== 0 && !searchInput" @click="upRow(j)">↑</div>
            <div v-show="j !== found.length - 1 && !searchInput" @click="downRow(j)">↓</div>
            <div @click="copyRow(j)">copy</div>
            <div @click="removeRow(j)">✖</div>
          </div>
          <td  class="table__cell" v-for="(td, k) in tr" :key="k" @click="editCell(j, k, $event)">
            {{ td }}
          </td>
        </tr>
      </tbody>
      <div v-if="!found.length">Not found</div>
    </table>
    <div v-if="error">{{ error }}</div>
  </div>
</template>

<script>
import { ref, computed } from '@vue/reactivity'
import readXlsxFile, { readSheetNames } from 'read-excel-file'
import { utils, writeFile } from 'xlsx'

import AppInputFile from '@/components/AppInputFile.vue'

import exportIcon from '@/assets/svg/exportIcon.svg'

export default {
  name: 'AppTable',
  components: { AppInputFile },
  setup() {
    // Импорт файла
    let file = null
    let fileName = null
    let sheetName = null
    let error = ref(null)
    let tableHead = ref(null)
    let tableBody = ref(null)
    function toImport(event) {
      if (event.target.files.length) {
        file = event.target.files[0]
        fileName = file.name
        readXlsxFile(file, { sheet: 1 }).then((rows) => {
          tableHead.value = rows[0]
          tableBody.value = [...rows]
          tableBody.value.shift()
        })
        readSheetNames(file).then((sheetNames) => {
          sheetName = sheetNames[0]
        })
        error.value = null
      } else {
        error.value = 'File not found'
      }
    }

    // Добавление строки
    function addRow() {
      let items = []
      tableHead.value.forEach(function() {
        items.push('')
      })
      tableBody.value.splice(0, 0, items)
    }
    // Удаление строки
    function removeRow(row) {
      tableBody.value.splice(row, 1)
    }
    // Поднять строку
    function upRow(row) {
      [tableBody.value[row], tableBody.value[row - 1]] = [tableBody.value[row - 1], tableBody.value[row]]
    }
    // Опустить строку
    function downRow(row) {
      [tableBody.value[row], tableBody.value[row + 1]] = [tableBody.value[row + 1], tableBody.value[row]]
    }
    // Копировать строку
    function copyRow(row) {
      tableBody.value.splice(row, 0, [...tableBody.value[row]])
    }

    // Показать методы редактирования
    function showMethods (event) {
      event.target.classList.add('active')
    }
    // Скрыть
    function hideMethods (event) {
      event.target.classList.remove('active')
    }

    // Редактирование ячейки
    const toEditCell = ref(null)
    let trActive = null
    let tdActive = null
    let editInput = ref(null)
    function change () {
      if(editInput.value) {
        trActive[tdActive] = editInput.value
        editInput.value = null
      }
      toEditCell.value.classList.remove('active')
    }
    // В теле таблицы
    function editCell(tr, td, event) {
      toEditCell.value.classList.add('active')
      event.target.appendChild(toEditCell.value)
      toEditCell.value.focus()
      trActive = tableBody.value[tr]
      tdActive = td
    }
    // В шапке
    function editHeadCell(td, event) {
      toEditCell.value.classList.add('active')
      event.target.appendChild(toEditCell.value)
      toEditCell.value.focus()
      trActive = tableHead.value
      tdActive = td
    }

    // Поиск
    let searchInput = ref(null)
    const found = computed(() => {
      if (searchInput.value) {
        const body = []
        tableBody.value.forEach(tr => {
          let flag = false
          tr.map(td => {
            if (td.toString().toLowerCase().includes(searchInput.value.toLowerCase())) flag = true
            return td
          })
          if (flag) body.push(tr)
        })
        return body
      } else {
        return tableBody.value
      }
    })

    // Экспорт файла
    function toExport () {
      let createXLSLFormatObj = []
      let newXlsHeader = []
      tableHead.value.map(th => {
          newXlsHeader.push(th);
      });
      createXLSLFormatObj.push(newXlsHeader);
      tableBody.value.map(tr => {
          let innerRowData = []
          tr.map(td => {
            innerRowData.push(td)
          });
          createXLSLFormatObj.push(innerRowData)
      });
      let wb = utils.book_new(),
          ws = utils.aoa_to_sheet(createXLSLFormatObj);
      utils.book_append_sheet(wb, ws, sheetName);
      writeFile(wb, fileName);
    }

    return {
      toImport, addRow, removeRow, upRow, downRow, copyRow, showMethods, hideMethods, change, editCell, editHeadCell, found, toExport,
      tableBody, tableHead, editInput, toEditCell, searchInput, error,
      exportIcon
    }
  }
}
</script>
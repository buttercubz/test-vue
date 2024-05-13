<script setup lang="ts">
import { FormInput, FormLabel } from "../base-components/Form";
import Litepicker from "../base-components/Litepicker";
import { ref, reactive, computed, watch } from "vue";
import { Tab } from "../base-components/Headless";
import Button from "../base-components/Button";
import Table from "../base-components/Table";
import Logo from "../assets/logo.png";

const items = ref<Array<{ item: string; cantidad: number; precio: number }>>(
  []
);

const backUp = JSON.parse(
  window.localStorage.getItem("factura.data")! ??
    `${JSON.stringify({
      fecha: new Date().toISOString(),
      numeroDefactura: "",
      representante: "",
      facturadoA: "",
      compañía: "",
      NCF: "",
      RNC: "",
      metodoDePago: "",
      informacionBancaria: "",
      nota: "",
    })}`
);

const currentItem = reactive({
  item: "",
  cantidad: 0,
  precio: 0,
});

const data = reactive({
  ...backUp,
});

watch(data, (value) => {
  window.localStorage.setItem("factura.data", JSON.stringify(value));
});

const date = computed(() => {
  const currentDate = new Date(data.fecha);

  //   // Format the date
  const formattedDate = new Intl.DateTimeFormat("en-GB", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
  }).format(currentDate);

  return formattedDate;
});

const formatDOP = (value: number) => {
  const formatter = new Intl.NumberFormat("es-DO", {
    style: "currency",
    currency: "DOP",
  });

  return formatter.format(value);
};

const SubTotal = computed(() => {
  const totalItems = items.value.reduce((prev, curr) => {
    return prev + curr.cantidad * curr.precio;
  }, 0);

  return totalItems;
});

const Itbis = computed(() => SubTotal.value * 0.18);

const Total = computed(() => SubTotal.value + Itbis.value);

const agregar = () => {
  items.value.push({ ...currentItem });

  currentItem.cantidad = 0;
  currentItem.precio = 0;
  currentItem.item = "";
};

const print = () => {
  const printContent = document.getElementById("preview")!.innerHTML;
  const originalContent = document.body.innerHTML;

  const originalTitle = document.head.title;

  document.body.innerHTML = printContent;
  document.head.title = "";

  document.title;

  window.print();

  document.body.innerHTML = originalContent;
  document.head.title = originalTitle;

  window.localStorage.setItem(
    "factura.data",
    `${JSON.stringify({
      fecha: new Date().toISOString(),
      numeroDefactura: "",
      representante: "",
      facturadoA: "",
      compañía: "",
      NCF: "",
      RNC: "",
      metodoDePago: "",
      informacionBancaria: "",
      nota: "",
    })}`
  );

  window.location.reload();
};
</script>

<template>
  <div class="flex flex-col">
    <div class="col-span-12 2xl:col-span-9">
      <div class="grid grid-cols-12 gap-6">
        <div class="col-span-12 mt-8 xl:col-span-9">
          <div class="flex items-center justify-between h-10 my-5">
            <h2 class="mr-5 text-lg font-medium truncate">Generar Factura</h2>
            <Button variant="primary" @click="print">Imprimir</Button>
          </div>
        </div>
      </div>
    </div>

    <div class="flex flex-col md:flex-row md:space-x-3">
      <div class="bg-white h-auto md:w-[80dvw] p-4 flex flex-col" id="editor">
        <Tab.Group>
          <Tab.List variant="tabs">
            <Tab>
              <Tab.Button class="w-full py-2" as="button">
                Encabezado
              </Tab.Button>
            </Tab>
            <Tab>
              <Tab.Button class="w-full py-2" as="button">
                Items a facturar
              </Tab.Button>
            </Tab>

            <Tab>
              <Tab.Button class="w-full py-2" as="button">
                Pie de factura
              </Tab.Button>
            </Tab>
          </Tab.List>

          <Tab.Panels>
            <Tab.Panel class="mt-10 space-y-4">
              <div>
                <FormLabel htmlFor="Numero-de-factura">
                  Numero de factura:</FormLabel
                >
                <FormInput
                  placeholder="Numero de factura"
                  v-model="data.numeroDefactura"
                  id="Numero-de-factura"
                  type="number"
                />
              </div>

              <div>
                <FormLabel htmlFor="NCF">NCF:</FormLabel>
                <FormInput
                  v-model="data.NCF"
                  placeholder="NCF"
                  type="text"
                  id="NCF"
                />
              </div>

              <div>
                <FormLabel htmlFor="fecha">Fecha:</FormLabel>
                <Litepicker
                  v-model="data.fecha"
                  :options="{
                    autoApply: false,
                    showWeekNumbers: true,
                    dropdowns: {
                      minYear: 2024,
                      maxYear: null,
                      months: true,
                      years: true,
                    },
                  }"
                />
              </div>

              <div>
                <FormLabel htmlFor="Facturado-a">Facturado a:</FormLabel>
                <FormInput
                  placeholder="Facturado a"
                  v-model="data.facturadoA"
                  id="Facturado-a"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="representante">Representante:</FormLabel>
                <FormInput
                  placeholder="Representante"
                  v-model="data.representante"
                  id="representante"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="Compañía">Compañía:</FormLabel>
                <FormInput
                  placeholder="Compañía"
                  v-model="data.compañía"
                  id="Compañía"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="RNC">RNC:</FormLabel>
                <FormInput
                  v-model="data.RNC"
                  placeholder="RNC"
                  type="text"
                  id="RNC"
                />
              </div>
            </Tab.Panel>

            <Tab.Panel class="mt-10 space-y-4">
              <div>
                <FormLabel htmlFor="item"> Item:</FormLabel>
                <FormInput
                  placeholder="Item"
                  v-model="currentItem.item"
                  id="item"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="cantidad"> Cantidad: </FormLabel>
                <FormInput
                  v-model="currentItem.cantidad"
                  placeholder="Cantidad"
                  id="cantidad"
                  type="number"
                  min="1"
                />
              </div>

              <div>
                <FormLabel htmlFor="precio">Precio DOP:</FormLabel>
                <FormInput
                  v-model="currentItem.precio"
                  placeholder="Precio DOP"
                  type="number"
                  id="precio"
                  min="1"
                />
              </div>

              <Button variant="primary" @click="agregar"> Agregar </Button>
            </Tab.Panel>

            <Tab.Panel class="mt-10 space-y-4">
              <div>
                <FormLabel htmlFor="metodoDePago"> Método de Pago:</FormLabel>
                <FormInput
                  placeholder="Método de Pago"
                  v-model="data.metodoDePago"
                  id="metodoDePago"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="informacionBancaria">
                  Informacion Bancaria:
                </FormLabel>
                <FormInput
                  v-model="data.informacionBancaria"
                  placeholder="Informacion Bancaria"
                  id="informacionBancaria"
                  type="text"
                />
              </div>

              <div>
                <FormLabel htmlFor="nota">Nota:</FormLabel>
                <FormInput
                  v-model="data.nota"
                  placeholder="Nota"
                  type="text"
                  id="nota"
                />
              </div>
            </Tab.Panel>
          </Tab.Panels>
        </Tab.Group>
      </div>

      <div
        class="bg-white h-[100dvh] md:w-[100dvw] flex flex-col items-center p-4 rounded overflow-y-auto"
        id="preview"
      >
        <div class="flex flex-row justify-between items-start w-full">
          <div />

          <div class="flex flex-col items-center">
            <img :src="Logo" alt="" class="w-40" />
            <span class="font-semibold">RNC: 132-77343-8</span>
          </div>

          <div />

          <div class="mt-[50px] absolute right-24">
            <span> NO. {{ data.numeroDefactura }} </span>
          </div>
        </div>

        <div class="w-full mt-5 px-10">
          <h1 class="text-2xl font-bold">FACTURA DE CRÉDITO FISCAL</h1>

          <span class="font-semibold uppercase">NCF: {{ data.NCF }}</span>
        </div>

        <div class="w-full mt-5 px-10 flex flex-col space-y-2">
          <span class="font-semibold">Fecha: {{ date }}</span>

          <span class="font-semibold">
            Facturado a:
            <span class="font-normal w-11/12 break-words whitespace-pre-wrap">
              {{ data.facturadoA }}
            </span>
          </span>

          <span class="font-semibold">
            Representante:

            <span class="font-normal w-11/12 break-words whitespace-pre-wrap">
              {{ data.representante }}
            </span>
          </span>

          <span class="font-semibold">
            Compañía:
            <span class="font-normal w-11/12 break-words whitespace-pre-wrap">
              {{ data.compañía }}
            </span>
          </span>

          <span class="font-semibold">
            RNC:
            <span class="font-normal w-11/12 break-words whitespace-pre-wrap">
              {{ data.RNC }}
            </span>
          </span>
        </div>

        <Table class="border-spacing-y-[10px] border-separate sm:mt-2 px-10">
          <Table.Thead class="bg-gray-200 !rounded">
            <Table.Tr>
              <Table.Th> Item </Table.Th>
              <Table.Th> Cantidad </Table.Th>
              <Table.Th> Precio DOP </Table.Th>
              <Table.Th> Total DOP </Table.Th>
            </Table.Tr>
          </Table.Thead>
          <Table.Tbody>
            <Table.Tr v-for="(prod, index) in [...items]" :key="index">
              <Table.Td> {{ prod.item }} </Table.Td>
              <Table.Td> {{ prod.cantidad }} </Table.Td>
              <Table.Td> {{ formatDOP(prod.precio) }} </Table.Td>
              <Table.Td>
                {{ formatDOP(prod.cantidad * prod.precio) }}
                <button
                  class="ml-5"
                  id="delete-item"
                  @click="items.splice(index, 1)"
                >
                  X
                </button>
              </Table.Td>
            </Table.Tr>
          </Table.Tbody>
        </Table>

        <div class="w-full flex justify-end mr-20 px-10">
          <div class="flex flex-col font-semibold text-end mt-3">
            <span>
              Sub-Total:
              <span class="font-normal">{{ formatDOP(SubTotal) || "" }}</span>
            </span>
            <span>
              ITBIS:
              <span class="font-normal">{{ formatDOP(Itbis) || "" }}</span>
            </span>
            <span>
              Total:
              <span class="font-normal">{{ formatDOP(Total) || "" }}</span>
            </span>
          </div>
        </div>

        <div class="flex flex-row w-full px-10 justify-between mt-5">
          <div class="flex flex-col">
            <span
              class="font-semibold max-w-[400px] break-words whitespace-pre-wrap"
            >
              Método de Pago:
              <span class="font-normal break-words whitespace-pre-wrap">
                {{ data.metodoDePago }}
              </span>
            </span>

            <span
              class="font-semibold max-w-[400px] break-words whitespace-pre-wrap"
            >
              Informacion Bancaria:
              <span class="font-normal break-words whitespace-pre-wrap">
                {{ data.informacionBancaria }}
              </span>
            </span>

            <span
              class="font-semibold max-w-[400px] break-words whitespace-pre-wrap"
            >
              Nota:
              <span class="font-normal break-words whitespace-pre-wrap">
                {{ data.nota }}
              </span>
            </span>
          </div>

          <div>
            <div>
              <span class="font-semibold">Correo: </span>
              negocios@softver-tech.com
            </div>

            <div>
              <span class="font-semibold">Web: </span>
              https://softver-tech.com.tech
            </div>
          </div>
        </div>

        <hr class="text-black" />
      </div>
    </div>
  </div>
</template>

<style>
@page {
  size: auto;
  margin: 0mm;
}

@media print {
  #delete-item {
    display: none;
  }
}
</style>

<script setup lang="ts">
// import TheWelcome from '../components/TheWelcome.vue'

import { ref } from 'vue'
import type { Ref } from 'vue';
import Swal from 'sweetalert2';

interface ProductInterface {
  id: number
  , name: string
  , description: string
  , price: number
}

interface OrderItemInterface {
  product: ProductInterface
  , quantity: number
  , subTotal: number
}

interface OrderInterface {
  items: OrderItemInterface[]
  , total: number
  , remark: string
}

const products: ProductInterface[] = [
  {
    "id": 1,
    "name": "珍珠奶茶",
    "description": "香濃奶茶搭配QQ珍珠",
    "price": 50
  },
  {
    "id": 2,
    "name": "冬瓜檸檬",
    "description": "清新冬瓜配上新鮮檸檬",
    "price": 45
  },
  {
    "id": 3,
    "name": "翡翠檸檬",
    "description": "綠茶與檸檬的完美結合",
    "price": 55
  },
  {
    "id": 4,
    "name": "四季春茶",
    "description": "香醇四季春茶，回甘無比",
    "price": 45
  },
  {
    "id": 5,
    "name": "阿薩姆奶茶",
    "description": "阿薩姆紅茶搭配香醇鮮奶",
    "price": 50
  },
  {
    "id": 6,
    "name": "檸檬冰茶",
    "description": "檸檬與冰茶的清新組合",
    "price": 45
  },
  {
    "id": 7,
    "name": "芒果綠茶",
    "description": "芒果與綠茶的獨特風味",
    "price": 55
  },
  {
    "id": 8,
    "name": "抹茶拿鐵",
    "description": "抹茶與鮮奶的絕配",
    "price": 60
  }
]

const order: Ref<OrderInterface> = ref({
  items: []
  , total: 0
  , remark: ""
})

const applyOrder: Ref<OrderInterface | null> = ref(null)

const addOrderItem = (product: ProductInterface) => {
  let isDuplicate = ref(false)
  order.value.items.forEach(item => {
    if (item.product.id === product.id) {
      Swal.fire({
        icon: 'error',
        title: 'Oops...',
        text: '已加入購物車'
      })
      isDuplicate.value = true
    }
  })
  if (isDuplicate.value) return

  order.value.items.push({
    product,
    quantity: 1,
    subTotal: product.price
  })

  reCalculateOrderTotal()
}

const removeOrderItem = (product: ProductInterface) => {
  order.value.items = order.value.items.filter(item => item.product.id !== product.id)
  reCalculateOrderTotal()
}

const reCalculateOrderTotal = () => {
  let total = 0
  order.value.items.forEach(item => {
    total += item.subTotal
  })
  order.value.total = total
}

const modifyOrderItemQuantity = (product: ProductInterface, quantity: number) => {
  order.value.items.forEach(item => {
    if (item.product.id === product.id) {
      item.quantity = quantity
      item.subTotal = item.product.price * quantity
    }
  })
  reCalculateOrderTotal()
}

const sendOrder = async () => {
  const userResult = await Swal.fire({
    icon: 'question',
    title: '確定要送出訂單嗎?',
    showDenyButton: true,
    // showCancelButton: true,
    confirmButtonText: '確定',
    denyButtonText: `取消`,
  })
  if (!userResult.isConfirmed) return

  applyOrder.value = { ...order.value }

  order.value = {
    items: []
    , total: 0
    , remark: ""
  }
}
</script>

<template>
  <div id="root">
    <div class="container mt-5">
      <div class="row">
        <div class="col-md-4">
          <div class="list-group">
            <a href="javascript:void(0)" class="list-group-item list-group-item-action" v-for="product in products"
              :key="product.id" @click="addOrderItem(product)">
              <div class="d-flex w-100 justify-content-between">
                <h5 class="mb-1">{{ product.name }}</h5>
                <small>${{ product.price }}</small>
              </div>
              <p class="mb-1">{{ product.description }}</p>
            </a>
          </div>
        </div>
        <div class="col-md-8">
          <template v-if="order.items.length > 0">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col" width="50">操作</th>
                  <th scope="col">品項</th>
                  <th scope="col">描述</th>
                  <th scope="col" width="90">數量</th>
                  <th scope="col">單價</th>
                  <th scope="col">小計</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in order.items" :key="item.product.id">
                  <td><button type="button" class="btn btn-sm" @click="removeOrderItem(item.product)">x</button></td>
                  <td>{{ item.product.name }}</td>
                  <td><small>{{ item.product.description }}</small></td>
                  <td>
                    <select class="form-select" v-model="item.quantity"
                      @change="modifyOrderItemQuantity(item.product, parseInt(($event.target as HTMLInputElement)!.value))">
                      <option value="1">1</option>
                      <option value="2">2</option>
                      <option value="3">3</option>
                      <option value="4">4</option>
                      <option value="5">5</option>
                      <option value="6">6</option>
                      <option value="7">7</option>
                      <option value="8">8</option>
                      <option value="9">9</option>
                      <option value="10">10</option>
                    </select>
                  </td>
                  <td>${{ item.product.price }}</td>
                  <td>${{ item.subTotal }}</td>
                </tr>
              </tbody>
            </table>
            <div class="text-end mb-3">
              <h5>總計: <span>${{ order.total }}</span></h5>
            </div>
            <textarea class="form-control mb-3" rows="3" placeholder="備註" v-model="order.remark"></textarea>
            <div class="text-end">
              <button class="btn btn-primary" @click="sendOrder">送出</button>
            </div>
          </template>
          <template v-else>
            <table class="table">
              <thead>
                <tr>
                  <th scope="col" width="50">操作</th>
                  <th scope="col">品項</th>
                  <th scope="col">描述</th>
                  <th scope="col" width="90">數量</th>
                  <th scope="col">單價</th>
                  <th scope="col">小計</th>
                </tr>
              </thead>
              <tbody>
              </tbody>
            </table>
            <div class="alert alert-primary text-center" role="alert"> 請選擇商品 </div>
          </template>
        </div>
      </div>
      <hr />
      <div class="row justify-content-center">
        <div class="col-8">
          <template v-if="applyOrder !== null">
            <div class="card">
              <div class="card-body">
                <div class="card-title">
                  <h5>訂單</h5>
                  <table class="table">
                    <thead>
                      <tr>
                        <th scope="col">品項</th>
                        <th scope="col">數量</th>
                        <th scope="col">小計</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="item in applyOrder.items" :key="item.product.id">
                        <td>{{ item.product.name }}</td>
                        <td>{{ item.quantity }}</td>
                        <td>${{ item.subTotal }}</td>
                      </tr>
                    </tbody>
                  </table>
                  <div class="text-end">備註: <span>{{ applyOrder.remark }}</span></div>
                  <div class="text-end">
                    <h5>總計: <span>${{ applyOrder.total }}</span></h5>
                  </div>
                </div>
              </div>
            </div>
          </template>
          <template v-else>
            <div class="alert alert-secondary text-center" role="alert"> 尚未建立訂單 </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

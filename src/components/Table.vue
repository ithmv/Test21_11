<template>
  <div>
    <table>
      <thead>
        <tr>
          <th @click="sort('o_id')">ID {{ getSortIndicator('o_id') }}</th>
          <th @click="sort('client_name')">Имя клиента {{ getSortIndicator('client_name') }}</th>
          <th @click="sort('diets')">Диеты {{ getSortIndicator('diets') }}</th>
          <th @click="sort('tariff')">Тариф {{ getSortIndicator('tariff') }}</th>
          <th @click="sort('dates.0.start_date')">Дата начала {{ getSortIndicator('dates.0.start_date') }}</th>
          <th @click="sort('dates.0.end_date')">Дата окончания {{ getSortIndicator('dates.0.end_date') }}</th>
          <th @click="sort('discount')">Скидка {{ getSortIndicator('discount') }}</th>
          <th @click="sort('order_sum')">Сумма заказа {{ getSortIndicator('order_sum') }}</th>
          <th @click="sort('order_payed')">Оплачен {{ getSortIndicator('order_payed') }}</th>
          <th @click="sort('pay_status')">Статус оплаты {{ getSortIndicator('pay_status') }}</th>
          <th>Комментарий курьера</th>
          <th>Внутренний комментарий</th>
          <th @click="sort('status')">Статус {{ getSortIndicator('status') }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(order, index) in sortedData" :key="index">
          <td>{{ order.o_id }}</td>
          <td>{{ order.client_name }}</td>
          <td>
            <span v-for="(diet, dietIndex) in order.diets" :key="dietIndex">
              {{ diet }}
              <br v-if="dietIndex !== order.diets.length - 1" />
            </span>
          </td>
          <td>
            <span v-for="(tariff, tariffIndex) in order.tariff" :key="tariffIndex">
              {{ tariff }}
              <br v-if="tariffIndex !== order.tariff.length - 1" />
            </span>
          </td>
          <td>{{ order.dates[0].start_date }}</td>
          <td>{{ order.dates[0].end_date }}</td>
          <td>{{ order.discount }}</td>
          <td>{{ order.order_sum }}</td>
          <td>{{ order.order_payed }}</td>
          <td>{{ order.pay_status }}</td>
          <td>{{ order.courier_comment }}</td>
          <td>{{ order.inner_comment }}</td>
          <td>{{ getOrderStatus(order) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import jsonData from '../data.json';

export default {
  data() {
    return {
      jsonData: jsonData.map(order => ({
        ...order,
        statusValue: this.calculateStatusValue(order.dates),
      })),
      sortColumn: '',
      sortOrder: 'asc',
    };
  },
  computed: {
    sortedData() {
      if (!this.sortColumn) return this.jsonData;

      return this.jsonData.slice().sort((a, b) => {
        const aValue = this.getNestedPropertyValue(a, this.sortColumn);
        const bValue = this.getNestedPropertyValue(b, this.sortColumn);

        if (this.sortColumn === 'status') {
          if (this.sortOrder === 'asc') {
            return a.statusValue - b.statusValue;
          } else {
            return b.statusValue - a.statusValue;
          }
        }

        if (this.sortOrder === 'asc') {
          return aValue > bValue ? 1 : -1;
        } else {
          return aValue < bValue ? 1 : -1;
        }
      });
    },
  },
  methods: {
    sort(column) {
      if (this.sortColumn === column) {
        this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortColumn = column;
        this.sortOrder = 'asc';
      }
    },
    getNestedPropertyValue(obj, path) {
      const keys = path.split('.');
      return keys.reduce((acc, key) => (acc && acc[key] ? acc[key] : ''), obj);
    },
    getSortIndicator(column) {
      if (this.sortColumn === column) {
        return this.sortOrder === 'asc' ? '▲' : '▼';
      }
      return '';
    },
    calculateStatusValue(dates) {
      const today = new Date();
      const endDate = new Date(dates[0].end_date);

      if (endDate > today) {
        return Math.floor((endDate - today) / (24 * 60 * 60 * 1000));
      } else {
        return -Math.floor((today - endDate) / (24 * 60 * 60 * 1000));
      }
    },
    getOrderStatus(order) {
      const days = order.statusValue;

      if (days > 0) {
        return `Заканчивается через ${days} дней`;
      } else if (days === 0) {
        return 'Заканчивается сегодня';
      } else {
        return `Закончилось ${-days} дней назад`;
      }
    },
  },
};
</script>





<style scoped lang="scss">
  table {
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
    background-color: #f2f2f2;
  }

  tr:nth-child(even) {
    background-color: #f9f9f9;
  }

  tr:hover {
    background-color: #e5e5e5;
  }
</style>
<script>
import Column from '@/components/Column.vue'
import Container from '@/components/Container.vue'
import DataTable from '@/components/DataTable.vue'
import Info from '@/components/Info.vue'
import NumberInput from '@/components/NumberInput.vue'
import Row from '@/components/Row.vue'
import StringSelect from '@/components/StringSelect.vue'

export default {
  components: {
    Column,
    Container,
    DataTable,
    Info,
    NumberInput,
    Row,
    StringSelect
  },
  computed: {
    instalmentAndPayment() {
      const that = this

      switch (this.type) {
        case 'fixed_instalments':
          return (interest, margin) => [that.fixedInstalment, that.fixedInstalment + interest + margin + that.recurringFee]
        case 'fixed_payments':
          return (interest, margin) => [that.fixedPayment - interest - margin - that.recurringFee, that.fixedPayment]
        case 'fixed_length_term':
          if (that.fixedTermLength === 0) {
            return null;
          }

          const repayments = Math.ceil(that.fixedTermLength / that.intervalLength)

          const intervalRate = that.intervalInterestRate + that.intervalMarginRate

          let sum;

          if (intervalRate === 0) {
            sum = that.principal / repayments
          } else {
            const compoundRate = (1 + intervalRate) ** repayments - 1

            sum = that.principal * (1 + compoundRate) * intervalRate / compoundRate
          }

          return (interest, margin) => [sum - interest - margin, sum + that.recurringFee]
        default:
          throw 'unknown type';
      }
    },
    intervalInterestRate() {
      return this.intervalLength * this.annualInterestPercentage / 12 / 100
    },
    intervalMarginRate() {
      return this.intervalLength * this.annualMarginPercentage / 12 / 100
    },
    payments() {
      if (this.principal === 0) {
        return [[0, 0, 0, 0, 0]]
      }

      if (this.intervalLength === 0) {
        return null
      }

      const instalmentAndPayment = this.instalmentAndPayment

      if (instalmentAndPayment === null) {
        return null
      }

      if (instalmentAndPayment(this.principal * this.intervalInterestRate, this.principal * this.intervalMarginRate)[0] <= this.principal * (this.intervalInterestRate + this.intervalMarginRate)) {
        return null
      }

      const payments = [[0, 0, this.initialFee, 0, this.initialFee]]

      let principal = this.principal

      while (principal >= 0.005) {
        let interest = principal * this.intervalInterestRate
        let margin = principal * this.intervalMarginRate

        let [instalment, payment] = instalmentAndPayment(interest, margin)

        if (instalment > principal) {
          instalment = principal
          payment = principal + interest + margin + this.recurringFee
        }

        payments.push([interest, margin, this.recurringFee, instalment, payment])

        principal -= instalment
      }

      return payments
    },
    totals() {
      const payments = this.payments

      if (payments === null) {
        return null
      } else {
        const totals = [0, 0, 0, 0, 0];

        for (let i = 0; i < payments.length; ++i) {
          for (let j = 0; j < totals.length; ++j) {
            totals[j] += payments[i][j];
          }
        }

        return totals
      }
    }
  },
  data() {
    return {
      principal: 0.00,
      intervalLength: 0,
      type: 'fixed_instalments',
      fixedInstalment: 0.00,
      fixedPayment: 0.00,
      fixedTermLength: 0,
      annualInterestPercentage: 0.0,
      annualMarginPercentage: 0.0,
      initialFee: 0.00,
      recurringFee: 0.00
    }
  }
}
</script>

<template>
  <Container>
    <h2>Loan</h2>
    <Row>
      <Column>
        <NumberInput description="Principal" :min="0" :step="0.01" :value="principal" unit="€" @update="value => principal = value"/>
      </Column>
      <Column>
        <NumberInput description="Interval length" :min="0" :max="12" :step="1" :value="intervalLength" unit="mos" @update="value => intervalLength = value"/>
      </Column>
    </Row>
    <Row>
      <Column>
        <StringSelect description="Type" :options="[{description: 'Fixed instalments', value: 'fixed_instalments'}, {description: 'Fixed payments', value: 'fixed_payments'}, {description: 'Fixed length term', value: 'fixed_length_term'}]" :value="type" @update="value => type = value"/>
      </Column>
      <Column>
        <NumberInput v-if="type === 'fixed_instalments'" description="Fixed instalment" :min="0" :step="0.01" :value="fixedInstalment" unit="€" @update="value => fixedInstalment = value"/>
        <NumberInput v-if="type === 'fixed_payments'" description="Fixed payment" :min="0" :step="0.01" :value="fixedPayment" unit="€" @update="value => fixedPayment = value"/>
        <NumberInput v-if="type === 'fixed_length_term'" description="Fixed term length" :min="0" :step="1" :value="fixedTermLength" unit="mos" @update="value => fixedTermLength = value"/>
      </Column>
    </Row>
    <Row>
      <Column>
        <NumberInput description="Annual interest percentage" :min="0.0" :step="0.1" :value="annualInterestPercentage" unit="%" @update="value => annualInterestPercentage = value"/>
      </Column>
      <Column>
        <NumberInput description="Annual margin percentage" :min="0.0" :step="0.1" :value="annualMarginPercentage" unit="%" @update="value => annualMarginPercentage = value"/>
      </Column>
    </Row>
    <Row>
      <Column>
        <NumberInput description="Recurring fee" :min="0" :step="0.01" :value="recurringFee" unit="€" @update="value => recurringFee = value"/>
      </Column>
      <Column>
        <NumberInput description="Initial fee" :min="0" :step="0.01" :value="initialFee" unit="€" @update="value => initialFee = value"/>
      </Column>
    </Row>
    <h2>Payments</h2>
    <div v-if="payments">
      <DataTable include-index head-index="Payment" :head="['Interest (€)', 'Margin (€)', 'Fee (€)', 'Instalment (€)', 'Payment (€)']" :body="payments" :format-body="value => value.toLocaleString([], {minimumFractionDigits: 2, maximumFractionDigits: 2})" foot-index="Totals" :foot="totals" :format-foot="value => value.toLocaleString([], {minimumFractionDigits: 2, maximumFractionDigits: 2})"/>
    </div>
    <div v-else>
      <Info>The loan is not soluble.</Info>
    </div>
  </Container>
</template>

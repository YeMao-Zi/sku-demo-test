<script setup lang="ts">
import { reactive, onMounted } from "vue";
import { getPrime, PathFinder, descartes } from "../src/sku/lib";

const dataSource = reactive({
  // 规格
  properties: [
    {
      name: "容量",
      attributes: [
        { value: "1L", isActive: false, isDisabled: false },
        { value: "4L", isActive: false, isDisabled: false },
      ],
    },
    {
      name: "颜色",
      attributes: [
        { value: "红色", isActive: false, isDisabled: false },
        { value: "黑色", isActive: false, isDisabled: false },
      ],
    },
    {
      name: "优惠套餐",
      attributes: [
        { value: "套餐一", isActive: false, isDisabled: false },
        { value: "套餐二", isActive: false, isDisabled: false },
      ],
    },
  ],
  selected: [], // 已经选中的规格
  selectedAttrs: [],
  unDisabled: [], // 可选规格
  skuList: [
    { id: "10", attributes: ["1L", "红色", "套餐一"] },
    { id: "20", attributes: ["1L", "黑色", "套餐二"] },
    { id: "30", attributes: ["4L", "红色", "套餐一"] },
    { id: "40", attributes: ["4L", "红色", "套餐二"] },
  ], // 可用sku
  valueInLabel: {}, // 质数，规格枚举值
});
let pathFinder: any;

const init = () => {
  // 获取全部规格
  const { skuList, properties } = dataSource;

  // 抹平规格内容
  const vertexList: any[] = [];
  properties.forEach((prop) => {
    prop.attributes.forEach((attr) => {
      vertexList.push(attr.value);
    });
  });
  // 通过抹平规格，获取规格对应质数
  const prime = getPrime(vertexList.length);
  // 质数对应规格数 枚举值处理
  const valueInLabel: Record<string, number> = {};
  vertexList.forEach((item, index) => {
    valueInLabel[item] = prime[index];
  });

  // 根据规格坐标，排序质数坐标
  const way = properties.map((i) => {
    return i.attributes.map((ii) => valueInLabel[ii.value]);
  });

  // 筛选可选的 SKU
  const _skuList = skuList.map((item) => {
    Reflect.set(
      item,
      "skuPrime",
      item.attributes.map((ii) => valueInLabel[ii])
    );
    // item.skuPrime = item.attributes.map((ii) => valueInLabel[ii]);
    return item;
  });

  // 初始化规格展示内容
  pathFinder = new PathFinder(
    way,
    _skuList.map((item) => item.skuPrime)
  );
  // 获取不可选规格内容
  const unDisabled = pathFinder.getWay().flat();

  const _properties = properties.map((item) => {
    item.attributes.forEach((attribute) => {
      attribute.isDisabled = !unDisabled.includes(valueInLabel[attribute.value]);
    });
    return item;
  });

  console.log(unDisabled, _skuList, vertexList, way, valueInLabel, _properties);

  dataSource.properties = _properties;
  dataSource.unDisabled = unDisabled;
  dataSource.valueInLabel = valueInLabel;
};

const handleClickAttribute = (propertyIndex: number, attributeIndex: number) => {
  // 获取已经选中的规格,质数，规格枚举值,以及原本规格名称
  const { selected, valueInLabel, properties } = dataSource;
  // 检查此次选择是否在已选内容中
  const attribute = properties[propertyIndex]["attributes"][attributeIndex];
  const type = attribute.value;
  const prime = Reflect.get(valueInLabel, type);
  const index = selected.indexOf(type);
  // 获取已经有的矩阵值
  const light = pathFinder.light;
  // 如果未选中则提供选中，如果选中移除
  if (index > -1) {
    pathFinder.remove(prime);
    selected.splice(index, 1);
  } else if (light[propertyIndex].includes(2)) {
    // 如果同规格中，有选中，则先移除选中，
    // 获取需要移除的同行规格
    const removeType = properties[propertyIndex]["attributes"].map((item) => item.value)[
      light[propertyIndex].indexOf(2)
    ];
    // 获取需要提出的同行规格质数
    const removePrime = Reflect.get(valueInLabel, removeType);
    // 移除
    pathFinder.remove(removePrime);
    selected.splice(selected.indexOf(removeType), 1);
    //移除同行后，添加当前选择规格
    pathFinder.add(prime);
    selected.push(type);
  } else {
    pathFinder.add(prime);
    selected.push(type);
  }

  // 更新不可选规格
  const unDisabled = pathFinder.getWay().flat();

  const _properties = properties.map((item) => {
    item.attributes.forEach((attribute) => {
      attribute.isDisabled = !unDisabled.includes(Reflect.get(valueInLabel, attribute.value));
      attribute.isActive = selected.includes(attribute.value);
    });
    return item;
  });
  console.log(_properties);

  dataSource.selected = selected;
  dataSource.unDisabled = unDisabled;
  dataSource.properties = _properties;
};

onMounted(() => {
  init();
});
</script>

<template>
  <div v-for="(item, propertyIndex) in dataSource.properties" :key="propertyIndex">
    <div>{{ item.name }}</div>
    <div class="attrbute">
      <div
        v-for="(attribute, attributeIndex) in item.attributes"
        :key="attributeIndex"
        @click="handleClickAttribute(propertyIndex, attributeIndex)"
        :class="[
          'weight',
          attribute.isActive ? 'seletedSpecifications' : '',
          attribute.isDisabled ? 'disabledStyle' : '',
        ]"
      >
        <div>{{ attribute.value }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.attrbute {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  flex: 1;
}

.weight {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  padding: 5px 20px;
  text-align: center;
  margin: 10rpx 0;
  background: #ffffff;
  border-radius: 10rpx;
  border: 2px solid #a6a6a6;
  font-size: 14px;
  color: rgba(0, 0, 0, 0.85);
  margin-right: 20rpx;
}

.disabledStyle {
  background-color: #f7f7f7;
}

.seletedSpecifications {
  border: 2px solid #fb6e23;
}
</style>

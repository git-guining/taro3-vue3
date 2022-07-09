<template>
	<view class="index">
		<view class="header" catch-move>
			<view class="tab_list">
				<view
					class="tab_list_item"
					:class="{ toggle: tabToggle === item.id, selected: tabSelected(item) }"
					v-for="item in tabList"
					:key="item.id"
					@tap="onClickTab(item)"
				>
					<view>{{ tabValue(item) }}</view>
				</view>
			</view>
			<view class="mask" :class="{ toggle: tabToggle }" @tap="closeTab"></view>
			<view class="patient_tab" :class="{ toggle: tabToggle === 1 }">
				<scroll-view class="patient_tab_list" :scroll-y="true">
					<view
						class="list_item"
						v-for="item in patientList"
						:key="item.id"
						@tap="curPatientTemporary = item"
					>
						<view class="item_left">
							<view class="name g-ellipsis">{{ item.name }}</view>
							<view class="sex_age">{{ item.sex === 1 ? '男' : '女' }}｜{{ item.age_desc }}</view>
						</view>
						<view class="item_right">
						</view>
					</view>
				</scroll-view>
				<view class="tab_bottom">
				</view>
			</view>
			<view class="status_tab" :class="{ toggle: tabToggle === 2 }">
				<view class="status_tab_list">
					<view
						class="list_item"
						:class="{ active: curStatusTemporary.id === item.id }"
						v-for="item in statusList"
						:key="item.id"
						@tap="curStatusTemporary = item"
					>{{ item.name }}</view>
				</view>
				<view class="tab_bottom">
				</view>
			</view>
		</view>

		<view class="pre_list">
			<view
				class="pre_list_item"
				:class="{ expired: item.state === 5 }"
				v-for="item in presList"
				:key="item.deal_id"
				@tap="toPresDetail(item)"
			>
				<view class="item_top">
					<view class="item_top_left">
						<view class="pre_type">{{ item.recipe_type_desc }}</view>
						<view class="expire_time" v-if="+item.expired_at">
							<text>有效期至 {{ getExpiredTime(item.expired_at) }}</text>
						</view>
					</view>
					<view class="item_top_right">{{ item.state_desc }}</view>
				</view>
				<view class="item_bottom">
					<view class="item_bottom_item">
						<view class="label">【 患者 】</view>
						<view
							class="value"
						>{{ item.patient_name }} | {{ item.patient_sex_desc }} | {{ item.patient_age_desc }}</view>
					</view>
					<view class="item_bottom_item">
						<view class="label">【 医生 】</view>
						<view class="value">{{ item.doctor_name }}</view>
					</view>
					<view class="item_bottom_item">
						<view class="label">【 辩证 】</view>
						<view class="value">{{ item.diagnose }}</view>
					</view>
					<view class="item_bottom_item">
						<view class="label">【 方案 】</view>
						<view class="value">{{ item.desc }}</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import { ref, onMounted, watch } from 'vue'
import Taro from '@tarojs/taro'
export default {
	components: {
	},
	props: {},
	emits: [],
	setup () {
		const tabList = ref([
			{ id: 1, name: '患者' },
			{ id: 2, name: '全部' },
		])
		const tabToggle = ref(0)
		const patientList = ref([])
		const curPatientTemporary = ref({})
		const curPatient = ref({})
		const statusList = ref([
			{ id: 0, name: '全部' },
			{ id: 2, name: '已购买' },
			{ id: 1, name: '未购买' },
			{ id: 5, name: '已过期' },
		])
		const curStatusTemporary = ref({ id: 0, name: '全部' })
		const curStatus = ref({ id: 0, name: '全部' })
		const pageNo = ref(1)
		const pageSize = 20
		const noMore = ref(false)
		const presList = ref([])

		const tabSelected = item => {
			if (item.id === 1 && curPatient.value.id) {
				return true
			}
			if (item.id === 2 && curStatus.value.id) {
				return true
			}
			return false
		}

		const tabValue = item => {
			if (item.id === 1 && curPatient.value.id) {
				return curPatient.value.name
			}
			if (item.id === 2 && curStatus.value.id) {
				return curStatus.value.name
			}
			return item.name
		}

		const onClickTab = ({ id }) => {
			if (tabToggle.value === id) {
				tabToggle.value = 0
			} else {
				tabToggle.value = id
			}
		}

		const onResetTab = () => {
			if (tabToggle.value === 1) {
				curPatientTemporary.value = {}
			}
			if (tabToggle.value === 2) {
				curStatusTemporary.value = { id: 0, name: '全部' }
			}
		}

		const onConfirmTab = async () => {
			if (tabToggle.value === 1) {
				if (curPatientTemporary.value.online_id === '') {
					const patientId = await createPatient(curPatientTemporary.value)
					curPatientTemporary.value.online_id = patientId
				}
				curPatient.value = curPatientTemporary.value
			}
			if (tabToggle.value === 2) {
				curStatus.value = curStatusTemporary.value
			}
			tabToggle.value = 0
		}

		const closeTab = () => {
			if (tabToggle.value === 1) {
				curPatientTemporary.value = curPatient.value
			}
			if (tabToggle.value === 2) {
				curStatusTemporary.value = curStatus.value
			}
			tabToggle.value = 0
		}

    const mydata = [{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1655856840","deal_id":"13854","desc":"陈皮12g,炒稻芽100g,金银花200g等50味药材","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022062200145531507","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"饮片方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1655856780","deal_id":"13853","desc":"陈皮12g,炒稻芽100g,金银花200g等37味药材","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022062200132880072","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"饮片方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1654769340","deal_id":"13530","desc":"陈皮12g,炒稻芽100g,金银花200g等12味药材","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022060910095258824","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"饮片方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1653736260","deal_id":"13210","desc":"陈皮12g,炒稻芽100g,金银花200g等12味药材","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022052811115570241","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"饮片方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652952660","deal_id":"13101","desc":"八二效全方4袋等2种","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051909312659611","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652952660","deal_id":"13100","desc":"八二效全方4袋等2种","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051909311947139","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652895120","deal_id":"13095","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051817322328949","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652290140","deal_id":"12975","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051117293844575","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652288280","deal_id":"12970","desc":"固本止痛片2袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116582610313","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":3,"recipe_type_desc":"中西成药","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652287860","deal_id":"12968","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116513739911","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652287200","deal_id":"12966","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116400890708","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652287020","deal_id":"12965","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116374546387","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652287020","deal_id":"12964","desc":"八二效全方2袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116370840163","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286960","deal_id":"12963","desc":"桂枝0.1g,炒酸枣仁9g","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116364045937","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"颗粒剂方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286960","deal_id":"12962","desc":"八二效全方3袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116361229800","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286900","deal_id":"12961","desc":"八二效全方3袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116355515882","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":4,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286900","deal_id":"12960","desc":"八二效全方3袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116353539087","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":3,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286540","deal_id":"12959","desc":"垂盆草72g,罗汉果23g,炒鸡内金5g等8味药材","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116292548426","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":3,"recipe_type_desc":"饮片方案","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286360","deal_id":"12958","desc":"八二效全方3袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116265037273","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":3,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"},{"address":"","area":"","cancel_time":"0","city":"","clinic_addr":"","clinic_id":0,"clinic_name":"","coupon_discount":0,"create_time":"1652286360","deal_id":"12957","desc":"八二效全方1袋","diagnose":"测试","doctor_id":"0","doctor_name":"桂宁","expire_day":0,"expired_at":"0","is_set_price":1,"online":1,"order_sn":"FA2022051116261527689","patient_age":0,"patient_age_desc":"30岁","patient_name":"可以","patient_sex":0,"patient_sex_desc":"男","pay_time":"0","pay_type":0,"price":0,"process_fee":0,"process_type_desc":"","province":"","receivables_price":0,"receiver_address":"","receiver_name":"","receiver_phone":"","recipe":[],"recipe_price":0,"recipe_status":3,"recipe_type_desc":"经验方","refund_fee":0,"refund_time":"0","ship_company":"","ship_fee":0,"ship_method":1,"ship_provider_serial":"","state":3,"state_desc":"已过期","stock_addr":"","take_code":0,"track_list":[],"user_id":"0"}]



		const getPatientList = async () => {

		}

		const createPatient = async patient => {
		}

		const getPresList = async () => {
      Taro.showLoading({ title: '加载中...'})
      console.log(new Date().getTime(), 'start')
			presList.value.push(...mydata)
      console.log(new Date().getTime(), 'end')
      setTimeout(() => {
        Taro.hideLoading()
      }, 100)
		}

		// 有效期问号弹窗
		const dialogShow = ref(false)
		const dialogContent = ref('')

		const onExpiredShow = item => {
			const day = item.online === 1 ? 3 : 2
			dialogContent.value = `以医生的开方时间起，${day}天内有效。截止时间为第${day}天的24:00；如有疑问请咨询在线客服～`
			dialogShow.value = true
		}

		const getExpiredTime = time => {
			return time
		}

		const toPresDetail = item => {

		};

		watch(
			[
				() => curPatient.value.id,
				() => curStatus.value.id,
			],
			() => {
				presList.value = []
				pageNo.value = 1
				noMore.value = false
				getPresList()
			}
		)

		onMounted(() => {
			// getPatientList()
			getPresList()
		})

		return {
			tabList,
			tabToggle,
			patientList,
			curPatientTemporary,
			curPatient,
			statusList,
			curStatusTemporary,
			curStatus,
			presList,
			noMore,
			tabSelected,
			tabValue,
			onClickTab,
			onResetTab,
			onConfirmTab,
			closeTab,
			getExpiredTime,
			getPresList,
			toPresDetail,
			dialogShow,
			dialogContent,
			onExpiredShow,
		}
	},
	onReachBottom () {
		this.getPresList()
	},
}
</script>

<style lang="scss">
.index {
	padding-bottom: calc(env(safe-area-inset-bottom) + 32px);
}
.header {
	position: sticky;
	top: 0;
	z-index: 10;
}
.tab_list {
	background: #ffffff;
	display: flex;
	height: 104px;
	padding: 28px 0;
	position: relative;
	z-index: 102;
	.tab_list_item {
		flex: 1;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 28px;
		line-height: 28px;
		.icon {
			margin-left: 4px;
			transition: transform 0.3s ease;
		}
	}
	.tab_list_item + .tab_list_item {
		border-left: 2px solid #e7e7e7;
	}
	.tab_list_item.toggle .icon {
		transform: rotate(180deg);
	}
	.tab_list_item.selected {
		color: $primaryColor;
		font-weight: 500;
	}
}
.pre_list {
	padding: 32px;
	padding-bottom: 0;
}
.pre_list_item {
	padding: 32px 32px 38px;
	background: #ffffff;
	border-radius: 24px;
	.item_top {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}
	.item_top_left {
		display: flex;
		align-items: center;
		.pre_type {
			height: 48px;
			background: #e6eed2;
			border-radius: 8px;
			padding: 0 16px;
			font-size: 28px;
			color: #65714b;
			line-height: 48px;
		}
		.expire_time {
			display: flex;
			align-items: center;
			font-size: 28px;
			color: #979797;
			line-height: 28px;
			margin-left: 24px;
			text {
				margin-right: 4px;
			}
		}
	}
	.item_top_right {
		font-size: 28px;
		font-weight: 500;
		color: $primaryColor;
		line-height: 28px;
	}
	.item_bottom {
		margin-top: 38px;
	}
	.item_bottom_item {
		display: flex;
		margin-left: -12px;
		font-size: 28px;
		line-height: 40px;
		.label {
			width: 136px;
			font-weight: 500;
			color: $primaryColor;
		}
		.value {
			flex: 1;
			color: $textColor2;
		}
	}
	.item_bottom_item + .item_bottom_item {
		margin-top: 16px;
	}
}
.pre_list_item + .pre_list_item {
	margin-top: 32px;
}
.pre_list_item.expired {
	position: relative;
	.pre_type {
		background: rgba(0, 0, 0, 0.09) !important;
		color: $textColor3 !important;
	}
	.expire_time,
	.item_top_right,
	.label,
	.value {
		color: #c1c1c1 !important;
	}
	.icon_overdue {
		position: absolute;
		right: 32px;
		bottom: 10px;
	}
}
.mask {
	position: fixed;
	top: 104px;
	left: 0;
	z-index: 100;
	width: 100%;
	height: 100%;
	opacity: 0;
	transform: scale3d(1, 1, 0);
	transition: all 0.3s ease;
	background: rgba(0, 0, 0, 0.6);
}
.mask.toggle {
	transform: scale3d(1, 1, 1);
	opacity: 1;
}
.patient_tab,
.status_tab {
	width: 100%;
	transform: translateY(calc(-100% - 104px));
	transition: transform 0.3s ease;
	position: absolute;
	z-index: 101;
	top: 104px;
}
.patient_tab.toggle,
.status_tab.toggle {
	transform: translateY(0);
}
.patient_tab_list {
	position: relative;
	z-index: 101;
	background: #ffffff;
	height: 364px;
	.list_item {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 40px 32px 40px 46px;
		.item_left {
			display: flex;
			align-items: baseline;
		}
		.name {
			font-size: 32px;
			font-weight: 600;
			line-height: 32px;
			max-width: 250px;
		}
		.sex_age {
			margin-left: 16px;
			font-size: 28px;
			color: $textColor2;
			line-height: 28px;
		}
	}
	.list_item:first-child {
		padding-top: 54px;
	}
	.list_item:last-child {
		padding-bottom: 54px;
	}
}
.tab_bottom {
	height: 144px;
	background: #ffffff;
	border-top: 1px solid $dividingLineColor;
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0 32px;
	.g-button {
		width: 336px;
		height: 80px;
		line-height: 80px;
	}
	.g-button + .g-button {
		margin-left: 22px;
	}
}
.status_tab_list {
	background: #ffffff;
	padding: 48px 32px;
	display: flex;
	flex-wrap: wrap;
	justify-content: space-between;
	.list_item {
		width: 212px;
		height: 56px;
		background: rgba(193, 193, 193, 0.2);
		border-radius: 8px;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28px;
		color: $textColor2;
		line-height: 28px;
	}
	.list_item:nth-child(n + 4) {
		margin-top: 24px;
	}
	.list_item.active {
		background: rgba(195, 146, 77, 0.2);
		color: $primaryColor;
		font-weight: 600;
	}
}
.expire-dialog {
	font-size: 28px;
	color: #666666;
	line-height: 40px;
}
</style>

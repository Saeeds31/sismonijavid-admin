<template>
    <div class="print-wrapper" v-if="checkPermission(['order_view'])">

        <!-- ================= TOOLBAR ================= -->
        <div class="print-toolbar">
            <div class="toolbar-content">
                <div class="toolbar-brand">
                    <div class="brand-mark overflow-hidden">
                        <img src="@/assets/images/logo.png" width="72" alt="">
                    </div>
                    <div>
                        <strong>سیسمونی جاوید</strong>
                        <span>مرکز مدیریت سفارشات</span>
                    </div>
                </div>

                <div class="toolbar-meta">
                    <div class="toolbar-chip">
                        <i class="bi bi-receipt"></i>
                        <span>{{ orders.length }} سفارش</span>
                    </div>
                    <div class="toolbar-chip">
                        <i class="bi bi-printer"></i>
                        <span>{{ printType === 'full' ? 'فاکتور سفارش' : 'برچسب ارسال' }}</span>
                    </div>
                </div>

                <div class="toolbar-actions">
                    <button @click="handlePrint" class="btn-print">
                        <i class="bi bi-printer-fill"></i>
                        <span>چاپ</span>
                    </button>
                    <router-link to="/orders" class="btn-back">
                        <i class="bi bi-arrow-right"></i>
                        <span>بازگشت</span>
                    </router-link>
                </div>
            </div>
        </div>

        <!-- ================= LOADING ================= -->
        <div v-if="loading" class="loading-state">
            <div class="loading-box">
                <div class="loading-spinner"></div>
                <strong>در حال آماده‌سازی سفارش‌ها</strong>
                <span>لطفاً چند لحظه صبر کنید...</span>
            </div>
        </div>

        <!-- ================= EMPTY ================= -->
        <div v-else-if="orders.length === 0" class="empty-state">
            <div class="empty-icon">
                <i class="bi bi-inbox"></i>
            </div>
            <h3>سفارشی برای نمایش وجود ندارد</h3>
            <p>لطفاً سفارش‌های مورد نظر را انتخاب کنید.</p>
            <router-link to="/orders" class="empty-button">
                <i class="bi bi-arrow-right"></i>
                بازگشت به سفارش‌ها
            </router-link>
        </div>

        <!-- ================= PRINT ================= -->
        <div id="printSection">

            <div class="print-body">

                <!-- ================= FULL PRINT ================= -->
                <template v-if="printType === 'full'">

                    <div class="invoice-page">
                        <div v-for="(order, index) in orders" :key="order.id" class="invoice-a4">

                            <!-- ===== باکس بالا: اطلاعات فروشگاه ===== -->
                            <div class="box box-store">
                                <div class="store-header">
                                    <div class="store-title">
                                        <strong>فروشگاه سیسمونی جاوید</strong>
                                    </div>
                                    <div class="store-date">
                                        <span>تاریخ چاپ:</span>
                                        <strong>{{ getCurrentDate() }}</strong>
                                    </div>
                                </div>

                                <div class="store-row">
                                    <div class="store-cell">
                                        <span class="cell-label">عنوان وبسایت:</span>
                                        <strong class="cell-value">sismonijavid.ir</strong>
                                    </div>
                                    <div class="store-cell">
                                        <span class="cell-label">شناسه سفارش:</span>
                                        <strong class="cell-value">#{{ String(order.id).padStart(5, '0') }}</strong>
                                    </div>
                                </div>

                                <div class="store-row">
                                    <div class="store-cell">
                                        <span class="cell-label">تلفن:</span>
                                        <strong class="cell-value">09022072235</strong>
                                    </div>
                                    <div class="store-cell">
                                        <span class="cell-label">شیوه ارسال:</span>
                                        <strong class="cell-value">{{ order?.shipping?.title }}</strong>
                                    </div>
                                </div>

                                <div class="store-row">
                                    <div class="store-cell">
                                        <span class="cell-label">کدپستی:</span>
                                        <strong class="cell-value">-------</strong>
                                    </div>
                                    <div class="store-cell full-width">
                                        <span class="cell-label">آدرس:</span>
                                        <strong class="cell-value">
                                            گلستان - گرگان - خیابان سرخواجه - مجتمع آفتاب 3 - طبقه همکف
                                        </strong>
                                    </div>
                                </div>
                            </div>

                            <!-- ===== باکس وسط: اطلاعات گیرنده ===== -->
                            <div class="box box-receiver">
                                <div class="receiver-header">
                                    <strong>اطلاعات گیرنده</strong>
                                </div>

                                <div class="receiver-grid">
                                    <div class="receiver-item">
                                        <span class="cell-label">گیرنده:</span>
                                        <strong class="cell-value">{{ order.user?.full_name ?? '-' }}</strong>
                                    </div>
                                    <div class="receiver-item">
                                        <span class="cell-label">کدپستی:</span>
                                        <strong class="cell-value">{{ order.address?.postal_code ?? '-' }}</strong>
                                    </div>
                                    <div class="receiver-item">
                                        <span class="cell-label">تلفن:</span>
                                        <strong class="cell-value">{{ order.user?.mobile ?? '-' }}</strong>
                                    </div>
                                    <div class="receiver-item">
                                        <span class="cell-label">تاریخ سفارش:</span>
                                        <strong class="cell-value">{{ new
                                            Date(order.created_at).toLocaleDateString('fa-IR') }}</strong>
                                    </div>
                                </div>

                                <div class="receiver-address">
                                    <span class="cell-label">آدرس:</span>
                                    <strong class="cell-value">
                                        {{ order.address?.province?.name ?? '' }} -
                                        {{ order.address?.city?.name ?? '' }} -
                                        {{ order.address?.address_line ?? '-' }}
                                    </strong>
                                </div>
                            </div>

                            <!-- ===== جدول آیتم‌ها ===== -->
                            <div class="box box-table">
                                <table class="items-table">
                                    <thead>
                                        <tr>
                                            <th class="col-index">#</th>
                                            <th class="col-product">محصول</th>
                                            <th class="col-specs">مشخصات</th>
                                            <th class="col-price">قیمت</th>
                                            <th class="col-discount">تخفیف</th>
                                            <th class="col-qty">تعداد</th>
                                            <th class="col-total">مبلغ کل</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <!-- آیتم‌های سفارش اصلی -->
                                        <tr v-for="(item, idx) in order.items" :key="item.id">

                                            <td class="cell-index">{{ String(idx + 1).padStart(2, '0') }}</td>
                                            <td class="cell-product">{{ item.product?.title ?? '-' }}</td>
                                            <td class="cell-specs">
                                                <template v-if="item.variant?.values?.length">
                                                    <span v-for="val in item.variant.values" :key="val.id"
                                                        class="spec-line">
                                                        {{ val.attribute ? val.attribute.name + ' ' : '' }}{{ val.value
                                                        }}
                                                    </span>
                                                </template>
                                                <span v-else class="muted">-</span>
                                            </td>
                                            <td class="cell-price">{{ Number(item.price).toLocaleString('fa-IR') }}</td>
                                            <td class="cell-discount">{{ Number(item.discount ??
                                                0).toLocaleString('fa-IR') }}</td>
                                            <td class="cell-qty">{{ item.quantity }}</td>
                                            <td class="cell-total">{{ Number(item.price *
                                                item.quantity).toLocaleString('fa-IR') }}</td>
                                        </tr>

                                        <!-- آیتم‌های سفارشات فرزند -->
                                        <template v-for="child in order.child_orders || []" :key="'child-' + child.id">
                                            <tr v-for="(item, cidx) in child.items" :key="item.id">
                                                <td class="cell-specs">
                                                    <template v-if="item.variant?.values?.length">
                                                        <span v-for="val in item.variant.values" :key="val.id"
                                                            class="spec-line">
                                                            {{ val.attribute ? val.attribute.name + ' ' : '' }}{{
                                                                val.value }}
                                                        </span>
                                                    </template>
                                                    <span v-else class="muted">-</span>
                                                </td>
                                                <td class="cell-index">
                                                    {{ String((order.items?.length || 0) + cidx + 1).padStart(2, '0') }}
                                                </td>
                                                <td class="cell-product">{{ item.product?.title ?? '-' }}</td>
                                                <td class="cell-price">{{ Number(item.price).toLocaleString('fa-IR') }}
                                                </td>
                                                <td class="cell-discount">{{ Number(item.discount ??
                                                    0).toLocaleString('fa-IR') }}</td>
                                                <td class="cell-qty">{{ item.quantity }}</td>
                                                <td class="cell-total">{{ Number(item.price *
                                                    item.quantity).toLocaleString('fa-IR') }}</td>
                                            </tr>
                                        </template>

                                        <tr v-if="!order.items?.length && !order.child_orders?.length">
                                            <td colspan="7" class="text-center muted">بدون آیتم</td>
                                        </tr>
                                    </tbody>
                                    <!-- ✅ ردیف تعداد کل -->
                                    <tfoot v-if="order.items?.length || order.child_orders?.length">
                                        <tr class="items-total-row">
                                            <td colspan="7">
                                                <strong>تعداد کل: {{ getTotalQuantity(order) }}</strong>
                                            </td>
                                        </tr>
                                    </tfoot>
                                </table>
                            </div>

                            <!-- ===== باکس جمع کل ===== -->
                            <div class="box box-summary">
                                <div class="summary-row">
                                    <div class="summary-cell">
                                        <span class="cell-label">مبلغ کل:</span>
                                        <strong class="cell-value">{{ Number(getSubTotal(order)).toLocaleString('fa-IR')
                                        }}
                                            تومان</strong>
                                    </div>
                                    <div class="summary-cell">
                                        <span class="cell-label">حمل و نقل:</span>
                                        <strong class="cell-value">{{
                                            Number(getShippingCost(order)).toLocaleString('fa-IR') }} تومان</strong>
                                    </div>
                                    <div class="summary-cell highlight">
                                        <span class="cell-label">مبلغ نهایی:</span>
                                        <strong class="cell-value">{{
                                            Number(getShippingCost(order) +getSubTotal(order)).toLocaleString('fa-IR') }}
                                            تومان</strong>
                                    </div>
                                </div>
                            </div>
                            <div class="box box-summary" v-if="order.user_note">
                                <div class="summary-row">
                                    <div class="summary-cell">
                                        <span class="cell-label">توضیحات سفارش:</span>
                                        <strong class="cell-value">{{ order.user_note }}
                                        </strong>
                                    </div>
                                </div>
                            </div>

                        </div>
                    </div>

                </template>

                <!-- ================= LABEL ================= -->
                <template v-else-if="printType === 'label'">

                    <div class="labels-grid">
                        <template v-for="order in orders" :key="order.id">
                            <div class="label-card">
                                <div class="label-top">
                                    <div class="label-brand">
                                        <div class="mini-logo overflow-hidden">
                                            <img src="@/assets/images/logo.png" width="72" alt="">
                                        </div>
                                        <div>
                                            <strong>اطلاعات فرستنده:</strong>
                                            <span>ماه ستی</span>
                                        </div>
                                    </div>
                                    <div class="label-number">
                                        <small>website</small>
                                        <strong>www.mahseti.shop</strong>
                                    </div>
                                </div>

                                <div class="label-route">
                                    <div class="route-title">
                                        <span class="route-line"></span>
                                        <i class="bi bi-arrow-down-circle-fill"></i>
                                        <span>ارسال از</span>
                                    </div>
                                    <div class="receiver-name">فروشگاه ماه ستی</div>
                                    <div class="receiver-phone">
                                        <i class="bi bi-telephone-fill"></i>
                                        ۰۹۳۳۵۸۱۴۴۷۵
                                    </div>
                                    <div class="receiver-address">
                                        آدرس: گلستان گرگان -سر سه راهی سلامتی
                                    </div>
                                    <div class="receiver-location">
                                        <span>گلستان - گرگان</span>
                                        <strong>کد پستی: ------</strong>
                                    </div>
                                </div>

                                <div class="label-bottom">
                                    <div>
                                        <small>mahseti</small>
                                        <strong>فروشگاه لباس زیر</strong>
                                    </div>
                                    <div class="label-price">
                                        <small>تاریخ سفارش</small>
                                        <strong>{{ new Date(order.created_at).toLocaleDateString('fa-IR') }}</strong>
                                    </div>
                                </div>
                            </div>

                            <div class="label-card">
                                <div class="label-top">
                                    <div class="label-brand">
                                        <div class="mini-logo">
                                            <i class="bi bi-bag-heart-fill"></i>
                                        </div>
                                        <div>
                                            <strong>اطلاعات گیرنده:</strong>
                                            <span>SHIPPING LABEL</span>
                                        </div>
                                    </div>
                                    <div class="label-number">
                                        <small>شناسه سفارش</small>
                                        <strong>#{{ String(order.id).padStart(5, '0') }}</strong>
                                    </div>
                                </div>

                                <div class="label-route">
                                    <div class="route-title">
                                        <span class="route-line"></span>
                                        <i class="bi bi-arrow-down-circle-fill"></i>
                                        <span>ارسال به</span>
                                    </div>
                                    <div class="receiver-name">{{ order.user?.full_name ?? '-' }}</div>
                                    <div class="receiver-phone">
                                        <i class="bi bi-telephone-fill"></i>
                                        {{ order.user?.mobile ?? '-' }}
                                    </div>
                                    <div class="receiver-address">
                                        {{ order.address?.address_line ?? '-' }}
                                    </div>
                                    <div class="receiver-location">
                                        <span>
                                            {{ order.address?.province?.name ?? '-' }} -
                                            {{ order.address?.city?.name ?? '-' }}
                                        </span>
                                        <strong>کد پستی: {{ order.address?.postal_code ?? '-' }}</strong>
                                    </div>
                                </div>

                                <div class="label-bottom">
                                    <div>
                                        <small>روش ارسال</small>
                                        <strong>{{ order.shipping?.title ?? '-' }}</strong>
                                    </div>
                                    <div class="label-price">
                                        <small>مبلغ سفارش</small>
                                        <strong>
                                            {{ Number(order.total).toLocaleString('fa-IR') }}
                                            <em>تومان</em>
                                        </strong>
                                    </div>
                                </div>
                            </div>
                        </template>
                    </div>

                </template>

            </div>

        </div>

    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import axios from "axios";
import { useAdmin } from '@/stores/modules/admin';

const route = useRoute();
const router = useRouter();
const store = useAdmin();
const checkPermission = store.checkPermission;

const orders = ref([]);
const loading = ref(false);
const orderIds = ref([]);
const printType = ref('full');

function getSubTotal(order) {
    if (!order) return 0;
    let childs = order.child_orders ? order.child_orders.filter(o => o.status == "paid") : [];
    let childTotal = childs.reduce((a, b) => {
        return a + b.subtotal
    }, 0)
    return childTotal + order.subtotal
}

function getShippingCost(order) {
    if (!order) return 0;
    let childs = order.child_orders ? order.child_orders.filter(o => o.status == "paid") : [];
    let childTotal = childs.reduce((a, b) => {
        return a + b.shipping_cost
    }, 0)
    return childTotal + order.shipping_cost
}

// ✅ تاریخ امروز به شمسی
const getCurrentDate = () => {
    return new Date().toLocaleDateString('fa-IR', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit'
    });
};

// ✅ تعداد کل آیتم‌های سفارش
const getTotalQuantity = (order) => {
    let total = 0;
    if (order?.items?.length) {
        total += order.items.reduce((sum, item) => sum + (Number(item.quantity) || 0), 0);
    }
    if (order?.child_orders?.length) {
        order.child_orders.forEach(child => {
            if (child?.items?.length) {
                total += child.items.reduce((sum, item) => sum + (Number(item.quantity) || 0), 0);
            }
        });
    }
    return total;
};

const handlePrint = () => {
    const printContents = document.getElementById('printSection').innerHTML;
    const fontUrl = `${window.location.origin}/fonts/yekanBakh/bold.ttf`;

    const iframe = document.createElement('iframe');
    iframe.style.position = 'fixed';
    iframe.style.right = '0';
    iframe.style.bottom = '0';
    iframe.style.width = '0';
    iframe.style.height = '0';
    iframe.style.border = 'none';

    document.body.appendChild(iframe);

    const doc = iframe.contentWindow.document;
    doc.open();
    doc.write(`
        <!DOCTYPE html>
        <html dir="rtl">
        <head>
            <meta charset="UTF-8">
            <title>پرینت سفارش‌ها</title>
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">
            <style>
                @font-face {
                    font-family: 'yekanbakhbold';
                    src: url('${fontUrl}') format('truetype');
                    font-weight: normal;
                    font-style: normal;
                    font-display: block;
                }

                * { margin: 0; padding: 0; box-sizing: border-box; }

                html, body {
                    direction: rtl;
                    font-family: 'yekanbakhbold', Tahoma, Arial, sans-serif !important;
                    background: #fff;
                    color: #000;
                    font-size: 10px;
                    line-height: 1.6;
                }

                body, body * {
                    font-family: 'yekanbakhbold', Tahoma, Arial, sans-serif !important;
                }

                /* ✅ سایز برگه: 8cm × 14cm */
                @page {
                    size: 80mm 140mm;
                    margin: 2mm;
                }

                .invoice-page {
                    display: block;
                }

                /* ✅ هر سفارش = یک صفحه کامل 8×14 با flex عمودی */
                .invoice-a4 {
                    width: 76mm;
                    height: 136mm;
                    overflow: hidden;
                    page-break-after: always;
                    break-after: page;
                    page-break-inside: avoid;
                    break-inside: avoid;
                    display: flex;
                    flex-direction: column;
                    gap: 2px;
                }

                .invoice-a4:last-child {
                    page-break-after: auto;
                    break-after: auto;
                }

                /* ===== باکس‌ها ===== */
                .box {
                    border: 0.7px solid #000;
                    page-break-inside: avoid;
                    break-inside: avoid;
                }

                /* ✅ جدول فضای باقی‌مانده رو پر می‌کنه */
                .box-table {
                    flex: 1;
                    display: flex;
                    flex-direction: column;
                    min-height: 0;
                    padding: 0;
                }

                .box-table .items-table {
                    flex: 1;
                    height: 100%;
                }

                /* ===== باکس فروشگاه ===== */
                .store-header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    padding: 5px 7px;
                    border-bottom: 0.7px solid #000;
                    background: #f5f5f5;
                }

                .store-title strong {
                    font-size: 11px;
                    font-weight: 900;
                }

                .store-date {
                    display: flex;
                    align-items: center;
                    gap: 3px;
                    font-size: 8px;
                }

                .store-date strong {
                    font-weight: 800;
                }

                .store-row {
                    display: flex;
                    padding: 4px 7px;
                    border-bottom: 0.7px solid #ddd;
                }

                .store-row:last-child {
                    border-bottom: none;
                }

                .store-cell {
                    display: flex;
                    align-items: center;
                    gap: 3px;
                    flex: 1;
                    padding: 0 3px;
                }

                .store-cell:not(:last-child) {
                    border-left: 0.7px solid #ddd;
                }

                /* ===== باکس گیرنده ===== */
                .receiver-header {
                    padding: 4px 7px;
                    background: #f5f5f5;
                    border-bottom: 0.7px solid #000;
                    font-size: 10px;
                }

                .receiver-header strong {
                    font-weight: 900;
                }

                .receiver-grid {
                    display: grid;
                    grid-template-columns: 1fr 1fr;
                    border-bottom: 0.7px solid #ddd;
                }

                .receiver-item {
                    display: flex;
                    align-items: center;
                    gap: 3px;
                    padding: 4px 7px;
                    border-bottom: 0.7px solid #ddd;
                }

                .receiver-item:nth-child(odd) {
                    border-left: 0.7px solid #ddd;
                }

                .receiver-item:nth-last-child(-n+2) {
                    border-bottom: none;
                }

                .receiver-address {
                    display: flex;
                    align-items: flex-start;
                    gap: 3px;
                    padding: 4px 7px;
                }

                /* ===== سلول‌ها ===== */
                .cell-label {
                    color: #555;
                    font-size: 8.5px;
                    font-weight: 600;
                    white-space: nowrap;
                }

                .cell-value {
                    font-weight: 800;
                    font-size: 10px;
                    word-break: break-word;
                }

                /* ===== جدول آیتم‌ها ===== */
                .items-table {
                    width: 100%;
                    border-collapse: collapse;
                    font-size: 9.5px;
                }

                .items-table thead {
                    background: #f5f5f5;
                }

                .items-table th {
                    border: 0.7px solid #000;
                    padding: 5px 2px;
                    font-weight: 900;
                    font-size: 9px;
                    text-align: center;
                    white-space: nowrap;
                }

                /* ✅ ارتفاع ردیف‌ها بیشتر بشه تا جدول پر بشه */
                .items-table td {
                    border: 0.7px solid #999;
                    padding: 7px 2px;
                    text-align: center;
                    vertical-align: middle;
                    font-size: 9.5px;
                    height: 24px;
                }

                .cell-specs {
                    text-align: right;
                    line-height: 1.6;
                    font-size: 8px;
                }

                .spec-line {
                    display: block;
                }

                .cell-product {
                    text-align: right;
                    font-weight: 700;
                }

                .cell-index {
                    color: #666;
                    font-weight: 700;
                }

                .cell-price,
                .cell-discount,
                .cell-total {
                    font-weight: 700;
                    white-space: nowrap;
                }

                .cell-total {
                    font-weight: 900;
                }

                .muted {
                    color: #999;
                }

                .text-center {
                    text-align: center;
                }

                /* ✅ ردیف تعداد کل */
                .items-total-row td {
                    background: #f5f5f5;
                    text-align: right;
                    padding: 5px 7px;
                    font-size: 10px;
                    border-top: 1.5px solid #000;
                }

                .items-total-row strong {
                    font-weight: 900;
                }

                /* ===== باکس جمع کل ===== */
                .summary-row {
                    display: flex;
                }

                .summary-cell {
                    flex: 1;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    gap: 3px;
                    padding: 7px 4px;
                    border-left: 0.7px solid #000;
                    text-align: center;
                }

                .summary-cell:last-child {
                    border-left: none;
                }

                .summary-cell.highlight {
                    background: #000 !important;
                    color: #fff !important;
                    -webkit-print-color-adjust: exact !important;
                    print-color-adjust: exact !important;
                    color-adjust: exact !important;
                }

                .summary-cell.highlight .cell-label {
                    color: #fff !important;
                    font-weight: 700;
                }

                .summary-cell.highlight .cell-value {
                    color: #fff !important;
                    font-size: 11px;
                    font-weight: 900;
                }

                /* ===== پرینت ===== */
                @media print {
                    .invoice-a4 {
                        margin: 0;
                    }

                    .box {
                        box-shadow: none;
                    }
                }
            </style>
        </head>
        <body>
            ${printContents}
        </body>
        </html>
    `);

    doc.close();

    setTimeout(() => {
        iframe.contentWindow.print();
        setTimeout(() => {
            document.body.removeChild(iframe);
        }, 1500);
    }, 500);
};

const fetchOrders = async () => {
    const ids = route.query.ids;
    const type = route.query.type || 'full';
    printType.value = type;

    if (!ids) {
        router.push('/orders');
        return;
    }

    orderIds.value = ids.split(',').map(id => parseInt(id));
    loading.value = true;

    try {
        const response = await axios.post('/orders/print-data', {
            ids: orderIds.value
        });

        if (response.data.success) {
            orders.value = response.data.data;
        } else {
            alert('خطا در دریافت اطلاعات سفارش‌ها');
            router.push('/orders');
        }
    } catch (error) {
        console.error('Error:', error);
        alert('خطا در ارتباط با سرور');
        router.push('/orders');
    } finally {
        loading.value = false;
    }
};

const formatDate = (date) => {
    if (!date) return '-';
    try {
        const d = new Date(date);
        return d.toLocaleDateString('fa-IR') + ' - ' + d.toLocaleTimeString('fa-IR', {
            hour: '2-digit',
            minute: '2-digit'
        });
    } catch {
        return date;
    }
};

const statusText = (status) => {
    const map = {
        pending: "در انتظار",
        reserved: "رزرو شده",
        processing: "در حال پردازش",
        paid: "پرداخت شده",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        returned: "مرجوعی",
    };
    return map[status] ?? status;
};

const statusClass = (status) => {
    const map = {
        pending: "status-pending",
        reserved: "status-reserved",
        processing: "status-processing",
        paid: "status-completed",
        shipped: "status-shipped",
        completed: "status-completed",
        canceled: "status-canceled",
        returned: "status-returned",
    };
    return map[status] ?? "status-pending";
};

const paymentStatusText = (status) => {
    const map = {
        pending: "در انتظار پرداخت",
        paid: "پرداخت شده",
        failed: "ناموفق",
        refunded: "برگشت داده شده",
    };
    return map[status] ?? status;
};

const paymentClass = (status) => {
    const map = {
        pending: "payment-pending",
        paid: "payment-paid",
        failed: "payment-failed",
        refunded: "payment-refunded",
    };
    return map[status] ?? "payment-pending";
};

const paymentMethodText = (method) => {
    const map = {
        online: "پرداخت آنلاین",
        wallet: "کیف پول",
        cod: "پرداخت در محل",
    };
    return map[method] ?? method;
};

onMounted(() => {
    fetchOrders();
});
</script>


<style scoped>
/* =========================================================
   BASE
========================================================= */
.print-wrapper {
    min-height: 100vh;
    background:
        radial-gradient(circle at top right, rgba(24, 24, 27, .035), transparent 30%),
        #f6f6f5;
    color: #18181b;
    direction: rtl;
}

.print-wrapper * {
    font-family: "yekanbakhbold" !important;
}

/* =========================================================
   TOOLBAR
========================================================= */
.print-toolbar {
    background: rgba(255, 255, 255, .92);
    backdrop-filter: blur(18px);
    border-bottom: 1px solid #e4e4e7;
}

.toolbar-content {
    max-width: 1400px;
    min-height: 76px;
    margin: auto;
    padding: 12px 28px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 25px;
}

.toolbar-brand {
    display: flex;
    align-items: center;
    gap: 11px;
}

.brand-mark {
    width: 42px;
    height: 42px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #18181b;
    color: #fff;
    border-radius: 12px;
    font-size: 18px;
}

.toolbar-brand strong {
    display: block;
    font-size: 14px;
    font-weight: 900;
}

.toolbar-brand span {
    display: block;
    color: #a1a1aa;
    font-size: 10px;
    margin-top: 2px;
}

.toolbar-meta {
    display: flex;
    align-items: center;
    gap: 8px;
}

.toolbar-chip {
    display: flex;
    align-items: center;
    gap: 7px;
    padding: 9px 13px;
    background: #fafafa;
    border: 1px solid #e4e4e7;
    border-radius: 10px;
    color: #52525b;
    font-size: 11px;
    font-weight: 700;
}

.toolbar-chip i {
    color: #18181b;
}

.toolbar-actions {
    display: flex;
    align-items: center;
    gap: 8px;
}

.btn-print,
.btn-back {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    height: 42px;
    padding: 0 18px;
    border-radius: 10px;
    font-size: 12px;
    font-weight: 800;
    cursor: pointer;
    text-decoration: none;
    transition: .2s ease;
}

.btn-print {
    border: 0;
    background: #18181b;
    color: #fff;
}

.btn-print:hover {
    background: #27272a;
    transform: translateY(-1px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, .12);
}

.btn-back {
    background: #fff;
    color: #52525b;
    border: 1px solid #e4e4e7;
}

.btn-back:hover {
    background: #f4f4f5;
    color: #18181b;
}

/* =========================================================
   PRINT SECTION
========================================================= */
#printSection {
    max-width: 1300px;
    margin: 0 auto;
    padding: 40px 30px 70px;
}

/* =========================================================
   LOADING
========================================================= */
.loading-state {
    min-height: 65vh;
    display: flex;
    align-items: center;
    justify-content: center;
}

.loading-box {
    width: 260px;
    padding: 30px;
    display: flex;
    flex-direction: column;
    align-items: center;
    background: #fff;
    border: 1px solid #e4e4e7;
    border-radius: 18px;
    box-shadow: 0 15px 45px rgba(0, 0, 0, .05);
}

.loading-spinner {
    width: 42px;
    height: 42px;
    border: 3px solid #e4e4e7;
    border-top-color: #18181b;
    border-radius: 50%;
    animation: spin .8s linear infinite;
    margin-bottom: 18px;
}

.loading-box strong {
    font-size: 13px;
}

.loading-box span {
    color: #a1a1aa;
    font-size: 10px;
    margin-top: 5px;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

/* =========================================================
   EMPTY
========================================================= */
.empty-state {
    min-height: 65vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.empty-icon {
    width: 75px;
    height: 75px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 22px;
    background: #fff;
    border: 1px solid #e4e4e7;
    color: #a1a1aa;
    font-size: 30px;
    margin-bottom: 20px;
}

.empty-state h3 {
    margin: 0;
    font-size: 17px;
    font-weight: 900;
}

.empty-state p {
    margin: 7px 0 20px;
    color: #a1a1aa;
    font-size: 12px;
}

.empty-button {
    display: inline-flex;
    align-items: center;
    gap: 7px;
    padding: 10px 16px;
    border-radius: 9px;
    background: #18181b;
    color: #fff;
    font-size: 11px;
    font-weight: 800;
    text-decoration: none;
}

/* =========================================================
   INVOICE (نمایش روی صفحه - با سایز 8×14)
========================================================= */
.invoice-page {
    display: flex;
    flex-direction: column;
    gap: 30px;
    align-items: center;
}

.invoice-a4 {
    background: #fff;
    border: 1px solid #e4e4e7;
    border-radius: 6px;
    padding: 8px;
    box-shadow: 0 12px 40px rgba(0, 0, 0, .05);
    width: 80mm;
    height: 140mm;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    gap: 2px;
}

/* ===== باکس‌ها ===== */
.invoice-a4 .box {
    border: 1px solid #000;
}

.invoice-a4 .box-table {
    flex: 1;
    display: flex;
    flex-direction: column;
    min-height: 0;
    padding: 0;
}

.invoice-a4 .box-table .items-table {
    flex: 1;
    height: 100%;
}

/* باکس فروشگاه */
.invoice-a4 .store-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 4px 6px;
    border-bottom: 1px solid #000;
    background: #f5f5f5;
}

.invoice-a4 .store-title strong {
    font-size: 10px;
    font-weight: 900;
}

.invoice-a4 .store-date {
    display: flex;
    align-items: center;
    gap: 3px;
    font-size: 8px;
}

.invoice-a4 .store-date strong {
    font-weight: 800;
}

.invoice-a4 .store-row {
    display: flex;
    padding: 3px 6px;
    border-bottom: 1px solid #ddd;
}

.invoice-a4 .store-row:last-child {
    border-bottom: none;
}

.invoice-a4 .store-cell {
    display: flex;
    align-items: center;
    gap: 3px;
    flex: 1;
    padding: 0 3px;
}

.invoice-a4 .store-cell:not(:last-child) {
    border-left: 1px solid #ddd;
}

/* باکس گیرنده */
.invoice-a4 .receiver-header {
    padding: 3px 6px;
    background: #f5f5f5;
    border-bottom: 1px solid #000;
    font-size: 9px;
}

.invoice-a4 .receiver-header strong {
    font-weight: 900;
}

.invoice-a4 .receiver-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    border-bottom: 1px solid #ddd;
}

.invoice-a4 .receiver-item {
    display: flex;
    align-items: center;
    gap: 3px;
    padding: 3px 6px;
    border-bottom: 1px solid #ddd;
}

.invoice-a4 .receiver-item:nth-child(odd) {
    border-left: 1px solid #ddd;
}

.invoice-a4 .receiver-item:nth-last-child(-n+2) {
    border-bottom: none;
}

.invoice-a4 .receiver-address {
    display: flex;
    align-items: flex-start;
    gap: 3px;
    padding: 4px 6px;
}

/* سلول‌ها */
.invoice-a4 .cell-label {
    color: #555;
    font-size: 8px;
    font-weight: 600;
    white-space: nowrap;
}

.invoice-a4 .cell-value {
    font-weight: 800;
    font-size: 9px;
    word-break: break-word;
}

/* جدول */
.invoice-a4 .items-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 8.5px;
}

.invoice-a4 .items-table thead {
    background: #f5f5f5;
}

.invoice-a4 .items-table th {
    border: 1px solid #000;
    padding: 4px 2px;
    font-weight: 900;
    font-size: 8px;
    text-align: center;
    white-space: nowrap;
}

.invoice-a4 .items-table td {
    border: 1px solid #999;
    padding: 6px 2px;
    text-align: center;
    vertical-align: middle;
    font-size: 8.5px;
    height: 22px;
}

.invoice-a4 .cell-specs {
    text-align: right;
    line-height: 1.5;
    font-size: 7.5px;
}

.invoice-a4 .spec-line {
    display: block;
}

.invoice-a4 .cell-product {
    text-align: right;
    font-weight: 700;
}

.invoice-a4 .cell-index {
    color: #666;
    font-weight: 700;
}

.invoice-a4 .cell-price,
.invoice-a4 .cell-discount,
.invoice-a4 .cell-total {
    font-weight: 700;
    white-space: nowrap;
}

.invoice-a4 .cell-total {
    font-weight: 900;
}

.invoice-a4 .muted {
    color: #999;
}

.invoice-a4 .text-center {
    text-align: center;
}

/* ✅ ردیف تعداد کل */
.invoice-a4 .items-total-row td {
    background: #f5f5f5;
    text-align: right;
    padding: 4px 6px;
    font-size: 9px;
    border-top: 2px solid #000;
}

.invoice-a4 .items-total-row strong {
    font-weight: 900;
}

/* جمع کل */
.invoice-a4 .summary-row {
    display: flex;
}

.invoice-a4 .summary-cell {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 3px;
    padding: 6px 3px;
    border-left: 1px solid #000;
    text-align: center;
}

.invoice-a4 .summary-cell:last-child {
    border-left: none;
}

.invoice-a4 .summary-cell.highlight {
    background: #000 !important;
    color: #fff !important;
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
    color-adjust: exact !important;
}

.invoice-a4 .summary-cell.highlight .cell-label {
    color: #fff !important;
    font-weight: 700;
}

.invoice-a4 .summary-cell.highlight .cell-value {
    color: #fff !important;
    font-size: 10px;
    font-weight: 900;
}

/* =========================================================
   LABELS (برچسب ارسال - سایز 8×14)
========================================================= */
.labels-grid {
    display: grid;
    grid-template-columns: repeat(2, 80mm);
    gap: 18px;
    justify-content: center;
}

.label-card {
    overflow: hidden;
    background: #fff;
    border: 1px solid #18181b;
    border-radius: 10px;
    box-shadow: 0 10px 35px rgba(0, 0, 0, .04);
    width: 80mm;
    height: 140mm;
    display: flex;
    flex-direction: column;
}

.label-top {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 6px 8px;
    background: #18181b;
    color: #fff;
    flex-shrink: 0;
}

.label-brand {
    display: flex;
    align-items: center;
    gap: 6px;
}

.mini-logo {
    width: 26px;
    height: 26px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid rgba(255, 255, 255, .25);
    border-radius: 7px;
    font-size: 11px;
}

.label-brand strong {
    display: block;
    font-size: 9px;
    font-weight: 900;
}

.label-brand span {
    display: block;
    color: #a1a1aa;
    font-size: 6px;
    letter-spacing: 1px;
    direction: ltr;
}

.label-number {
    text-align: left;
}

.label-number small {
    display: block;
    color: #a1a1aa;
    font-size: 6px;
    direction: ltr;
}

.label-number strong {
    font-size: 11px;
}

.label-route {
    padding: 8px;
    flex: 1;
}

.route-title {
    display: flex;
    align-items: center;
    gap: 5px;
    color: #71717a;
    font-size: 7px;
    font-weight: 700;
    margin-bottom: 5px;
}

.route-title i {
    color: #18181b;
}

.route-line {
    width: 20px;
    height: 1px;
    background: #d4d4d8;
}

.receiver-name {
    font-size: 12px;
    font-weight: 900;
    margin-bottom: 3px;
}

.receiver-phone {
    color: #52525b;
    font-size: 7px;
    margin-bottom: 6px;
}

.receiver-phone i {
    margin-left: 3px;
}

.receiver-address {
    padding: 6px;
    background: #fafafa;
    border: 1px solid #f0f0f0;
    border-radius: 6px;
    font-size: 7px;
    line-height: 1.7;
}

.receiver-location {
    display: flex;
    justify-content: space-between;
    gap: 8px;
    margin-top: 5px;
    color: #52525b;
    font-size: 7px;
}

.receiver-location strong {
    color: #18181b;
    font-weight: 900;
}

.label-bottom {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 6px 8px;
    background: #fafafa;
    border-top: 1px dashed #d4d4d8;
    flex-shrink: 0;
}

.label-bottom small {
    display: block;
    color: #a1a1aa;
    font-size: 6px;
}

.label-bottom strong {
    display: block;
    font-size: 8px;
    font-weight: 900;
}

.label-price {
    text-align: left;
}

.label-price strong {
    font-size: 10px;
}

.label-price em {
    color: #71717a;
    font-size: 6px;
    font-style: normal;
}

/* =========================================================
   RESPONSIVE
========================================================= */
@media (max-width: 1000px) {
    .toolbar-content {
        padding: 12px 18px;
    }

    .toolbar-meta {
        display: none;
    }

    #printSection {
        padding: 25px 18px 50px;
    }

    .labels-grid {
        grid-template-columns: 80mm;
    }
}

@media (max-width: 650px) {
    .toolbar-brand {
        display: none;
    }

    .toolbar-content {
        justify-content: space-between;
    }

    .label-bottom,
    .receiver-location {
        flex-direction: column;
        align-items: flex-start;
    }

    .label-price {
        text-align: right;
    }
}

/* =========================================================
   PRINT SCREEN
========================================================= */
@media print {

    .print-wrapper {
        background: #fff;
    }

    .print-toolbar {
        display: none !important;
    }

    #printSection {
        max-width: none;
        padding: 0;
        margin: 0;
    }

    .invoice-page {
        display: block;
        gap: 0;
    }

    .invoice-a4 {
        background: none;
        border: none;
        border-radius: 0;
        padding: 0;
        box-shadow: none;
        max-width: none;
        margin: 0;
        width: 76mm;
        height: 136mm;
        overflow: hidden;
        page-break-after: always;
        break-after: page;
    }

    .invoice-a4:last-child {
        page-break-after: auto;
        break-after: auto;
    }

    .labels-grid {
        display: block;
        gap: 0;
    }

    .label-card {
        width: 76mm;
        height: 136mm;
        margin: 0 auto;
        page-break-after: always;
        break-after: page;
        border: 0.5px solid #000;
        border-radius: 0;
        box-shadow: none;
    }

    .label-card:last-child {
        page-break-after: auto;
        break-after: auto;
    }
}
</style>
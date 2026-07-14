---
id: assigning-seats
title: 分配订阅席位
description: "本分步指南将向你展示如何通过 Avalonia 门户为组织成员分配席位。"
doc-type: how-to
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
  - xpf
---

import PortalLogin from '/img/tools/assigning-seats/1-portal-login.png';
import SelectSubscription from '/img/tools/assigning-seats/2-select-subscription.png';
import GoToManagement from '/img/tools/assigning-seats/3-go-to-management.png';
import ClickAssignSeats from '/img/tools/assigning-seats/4-click-assign-seats.png';
import AssignModal from '/img/tools/assigning-seats/5-assign-modal.png';

你的 Avalonia 订阅包含一定数量的席位，在组织成员可以使用 Avalonia Plus、Pro、Enterprise 或 XPF 的高级功能之前，必须将这些席位分配给他们。本分步指南将向你展示如何为用户分配席位。

席位分配对 [Avalonia](https://avaloniaui.net/pricing) 和 [XPF](/xpf) 订阅的工作方式相同。

**你必须是组织的管理员才能分配席位。**

## 登录 Avalonia 门户

<table>
  <tbody>
    <tr>
      <td>1. 前往 Avalonia 门户：[https://portal.avaloniaui.net/](https://portal.avaloniaui.net/)。</td>
      <td rowspan="2"><Image light={PortalLogin}/></td>
    </tr>
    <tr>
      <td>2. 使用你的凭据登录。</td>
    </tr>
  </tbody>
</table>

## 进入订阅管理页面

<table>
  <tbody>
    <tr>
      <td>3. 在首页的磁贴中点击相关的订阅方案。</td>
      <td><Image light={SelectSubscription}/></td>
    </tr>
    <tr>
      <td>4. 在订阅页面右上角点击 **Manage Subscription**。</td>
      <td><Image light={GoToManagement}/></td>
    </tr>
  </tbody>
</table>

## 为用户分配席位

<table>
  <tbody>
    <tr>
      <td>5. 向下滚动订阅管理页面，直到找到标题为"Assigned Seats"的部分。</td>
      <td rowspan="2"><Image light={ClickAssignSeats}/></td>
    </tr>
    <tr>
      <td>6. 点击 **Assign Seats**。</td>
    </tr>
    <tr>
      <td>7. 在选择弹窗中，勾选你想要分配席位的用户。</td>
      <td rowspan="2"><Image light={AssignModal}/></td>
    </tr>
    <tr>
      <td>8. 点击 **Assign**。</td>
    </tr>
  </tbody>
</table>

:::warning
席位一旦分配，无法取消分配或重新分配。

如果你认为自己错误地将席位分配给了错误的用户，请在[门户](https://portal.avaloniaui.net/)中提交工单。
:::

## 另请参阅
- [登录问题故障排查](/troubleshooting/login-issues)
- [Avalonia XPF](/xpf)

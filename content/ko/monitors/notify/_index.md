---
aliases:
- /ko/monitors/faq/how-do-i-add-custom-template-variables-to-my-monitor-message
- /ko/monitors/faq/how-do-i-setup-conditional-contacts-and-messages-in-a-single-monitor
- /ko/developers/faq/what-do-notifications-do-in-datadog
- /ko/monitors/notifications/
description: 모니터가 알림을 트리거하면 팀에 알림을 전송하세요.
further_reading:
- link: /monitors/
  tag: 설명서
  text: 모니터 생성
- link: /monitors/manage/
  tag: 설명서
  text: 모니터 관리
kind: 설명서
title: 알림
---

## 개요

알림은 팀에게 문제를 알리고 문제 해결을 지원하는 모니터의 핵심 구성 요소입니다. [모니터 생성][1] 시 **Say what's happening** 및 **Notify your team** 섹션에 추가하세요.

## Say what's happening

이 섹션에서 팀에 보내는 알림을 설정할 수 있습니다.

### 타이틀

모니터에 고유한 제목을 추가합니다(필수). 다중 알림 모니터의 경우 트리거 범위를 식별하는 일부 태그가 자동으로 삽입됩니다. 또한 [태그 변수][2]를 사용할 수도 있습니다.

### 메시지

메시지 필드에는 표준 [마크다운 서식][3] 및 [변수][4]를 사용할 수 있습니다. [조건부 변수][5]를 사용하면[@notifications](#notifications)를 통해 여러 연락처로 전송되는 알림 텍스트를 설정할 수 있습니다.

모니터 메시지의 일반적인 사용 사례는 다음과 같이 단계별 문제 해결 방법을 포함하는 것입니다:

```text
디스크 용량 확보 방법:
1. 사용하지 않는 패키지 삭제
2. APT 캐시 삭제
3. 불필요한 애플리케이션 삭제
4. 중복 파일 삭제
```

### 태그

모니터에 태그를 추가합니다. 모니터 태그는 메트릭 태그와 다릅니다. UI에서 모니터를 그룹화하고 검색하는 데 사용됩니다. 태그 정책이 설정된 경우 필요한 태그와 태그 값을 추가해야 합니다. 자세한 내용은 [태그 정책][6]을 참조하세요.

{{< img src="monitors/notifications/notifications_add_required_tags.png" alt="정책 태그 설정 보기. '정책 태그' 아래에는 'Select value' 드롭다운 메뉴 옆에 cost_center, product_id, env라는 세 가지 예시 태그가 있습니다." style="width:100%;" >}}

### 재알림

모니터 재알림(선택 사항)을 활성화하여 문제가 해결되지 않았음을 팀에게 알릴 수 있습니다.

  {{< img src="monitors/notifications/renotify_options.png" alt="재알림 활성화" style="width:90%;" >}}

재알림 간격, 모니터가 재알림을 보내는 모니터 상태(`alert`, `no data`, `warn`)를 설정하고 필요 시 재알림 메시지 전송 횟수에 대한 제한을 설정합니다.

예를 들어, 기본 알림 후 에스컬레이션 메시지를 한 번만 수신하도록 모니터를 `stop renotifying after 1 occurrence`로 설정합니다.
**참고: 재알림의 [속성 및 태그 변수][7]는 재알림 기간 동안 모니터에서 사용할 수 있는 데이터로 채워집니다.

재알림을 활성화하면 모니터가 지정된 기간 동안 선택한 상태 중 하나로 유지될 경우 에스컬레이션 메시지를 전송하는 옵션이 제공됩니다.


에스컬레이션 메시지는 다음과 같은 방법으로 추가할 수 있습니다:

* 원래 알림 메시지의 `{{#is_renotify}}` 블록에서 추가합니다 (권장).
* `Say what's happening` 섹션의 *Renotification message* 필드에서 추가합니다.
* API의 `escalation_message` 속성을 사용합니다.

`{{#is_renotify}}` 블록을 사용하면 원래 알림 메시지도 재알림에 포함됩니다. 그렇기 때문에:

1. `{{#is_renotify}}` 블록에는 추가 정보만 포함하고 원래 메시지에 있던 세부 정보는 추가하지 마세요.
2. 그룹의 하위 집합에 에스컬레이션 메시지를 보냅니다.

[예제 섹션][8]에서 이러한 사용 사례에 맞게 모니터를 설정하는 방법을 알아보세요.

### 우선순위

모니터와 관련된 우선순위(선택 사항)를 추가합니다. 값의 범위는 P1에서 P5까지이며, P1이 가장 높은 우선순위이고 P5가 가장 낮은 우선순위입니다.
알림 메시지에서 모니터 우선순위를 재정의하려면 `Pi`가 P1과 P5 사이에 있는 `{{override_priority 'Pi'}}`를 사용합니다.

예를 들어, `alert`와 `warning`알림에 대해 다른 우선순위를 설정할 수 있습니다:

```
{{#is_alert}}
{{override_priority 'P1'}}
 ...
{{/is_alert}}

{{#is_warning}}
{{override_priority 'P4'}}
...
{{/is_warning}}
```

## 팀에 알림 전송하기

이 섹션을 사용하여 이메일, Slack, PagerDuty 등을 통해 팀에 알림을 보낼 수 있습니다. 드롭다운 상자를 사용하여 팀원 및 연결된 통합을 검색할 수 있습니다. 이 섹션에 `@notification`을 추가하면 [message](#message) 필드에 알림이 자동으로 추가됩니다.

**참고**: 예를 들어 `@notification`과 마지막 줄 문자 사이에는 다음과 같이 반드시 공백이 있어야 합니다:

```text
디스크 공간 부족 @ops-team@company.com
```

### 알림

`@notifications`는 다음으로 전송됩니다:

#### 이메일

{{% notifications-email %}}

#### 통합

{{% notifications-integrations %}}

### 추가 콘텐츠 토글

모니터 알림에는 모니터의 쿼리, 사용된 @-mentions, 메트릭 스냅샷(메트릭 모니터의 경우), Datadog의 관련 페이지로 돌아가는 링크 등의 콘텐츠가 포함됩니다. 개별 모니터에 대한 알림에서 어떤 콘텐츠를 포함하거나 제외할지 선택할 수 있습니다.

{{< img src="monitors/notifications/monitor_notification_presets.png" alt="모니터 프리셋 설정" style="width:70%;" >}}

옵션은 다음과 같습니다:

- **Default**: 숨겨진 콘텐츠가 없습니다.
- **Hide Query**: 알림 메시지에서 모니터의 쿼리를 제거합니다.
- **Hide Handles**: 알림 메시지에 사용된 @-mentions를 제거합니다.
- **Hide All**: 알림 메시지에는 쿼리, 핸들, 스냅샷(메트릭 모니터의 경우) 또는 바닥글에 있는 추가 링크가 포함되지 않습니다.

**참고**: 통합에 따라 일부 콘텐츠는 기본적으로 표시되지 않을 수 있습니다.

### 수정 사항

[이벤트][9]는 모니터가 만들어지거나, 수정되거나, 뮤트되거나, 삭제될 때마다 만들어집니다. 이러한 이벤트를 팀원, 채팅 서비스, 모니터 작성자에게 알리도록 `Notify` 옵션을 설정합니다.

### 권한 허용

모든 사용자는 연결된 역할에 관계없이 모든 모니터를 읽을 수 있습니다.

기본적으로 [모니터 쓰기 권한][10]이 있는 역할을 가진 사용자만 모니터를 편집할 수 있습니다. [Datadog 관리자 역할 및 Datadog 표준 역할][11]에는 기본적으로 모니터 쓰기 권한이 있습니다. 조직에서 [커스텀 역할][12]을 사용하는 경우 다른 커스텀 역할에 모니터 쓰기 권한이 있을 수 있습니다.

모니터를 수정할 수 있는 [역할][13] 목록을 지정하여 모니터를 더욱 제한할 수 있습니다. 모니터를 만든 사람은 항상 모니터를 편집할 수 있습니다.

  {{< img src="monitors/notifications/monitor_rbac_restricted.jpg" alt="RBAC 제한된 모니터" style="width:90%;" >}}

편집에는 모니터 설정 업데이트, 모니터 삭제, 원하는 시간 동안 모니터 뮤트하기 등이 포함됩니다.

**참고**: 제한 사항은 UI와 API 모두에 적용됩니다.

모니터의 RBAC 설정과 모니터를 고정 설정에서 역할 제한 사용으로 마이그레이션하는 방법에 대한 자세한 내용은 [모니터용 RBAC 설정 방법][14]을 참조하세요.

## 알림 테스트하기

테스트 알림은 [모니터 유형][15]에 대해 지원됩니다: 호스트, 메트릭, 이상, 이상치, 예측, 로그, rum, apm, 통합(검사 전용), 프로세스(검사 전용), 네트워크(검사 전용), 커스텀 검사, 이벤트 및 컴포지트.

### 테스트 실행하기

1. 모니터를 정의한 후 모니터 페이지 오른쪽 하단의 **Test Notifications** 버튼으로 알림을 테스트합니다.

2. 테스트 알림 팝업에서 테스트할 모니터 사례를 선택합니다. 경고 조건에 지정된 임계값에 대해 모니터 설정에서 사용할 수 있는 상태만 테스트할 수 있습니다. 모니터가 더 이상 경고 상태가 아니거나 경고 조건이 없는 경우 Datadog이 복구 알림을 보내기 때문에 [복구 임계값][16]은 예외입니다. 

    {{< img src="monitors/notifications/test-notif-select.png" alt="이 모니터에 대한 알림 테스트" style="width:70%;" >}}

3. **Run Test**를 클릭하면 모니터 목록에 있는 팀원 및 서비스에 알림을 보낼 수 있습니다.

### Events

테스트 알림은 이벤트 탐색기에서 검색할 수 있는 이벤트를 생성합니다. 이러한 알림은 알림 제목의 `[TEST]` 및 메시지 본문에 누가 테스트를 시작했는지 표시합니다.

태그 변수는 Datadog 하위 이벤트의 텍스트에만 채워집니다. 상위 이벤트는 집계 요약만 표시합니다.

### 변수 {#variables-test-notification}

메시지 변수는 모니터 정의 범위에 따라 무작위로 선택된 그룹으로 자동으로 채워집니다. 예를 들어:

```text
{{#is_alert}}
{{host.name}} <-- 여기에 생성
{{/is_alert}}
```
## 참고 자료

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ko/monitors/configuration
[2]: /ko/monitors/notify/variables/#tag-variables
[3]: http://daringfireball.net/projects/markdown/syntax
[4]: /ko/monitors/notify/variables/
[5]: /ko/monitors/notify/variables/#conditional-variables
[6]: /ko/monitors/settings/
[7]: /ko/monitors/notify/variables/?tabs=is_alert#attribute-and-tag-variables
[8]: /ko/monitors/notify/variables/?tab=is_renotify#examples
[9]: /ko/events/
[10]: /ko/account_management/rbac/permissions/#monitors
[11]: /ko/account_management/rbac/?tab=datadogapplication#datadog-default-roles
[12]: /ko/account_management/rbac/?tab=datadogapplication#custom-roles
[13]: /ko/account_management/rbac/?tab=datadogapplication
[14]: /ko/monitors/guide/how-to-set-up-rbac-for-monitors/
[15]: /ko/monitors/types
[16]: /ko/monitors/guide/recovery-thresholds/
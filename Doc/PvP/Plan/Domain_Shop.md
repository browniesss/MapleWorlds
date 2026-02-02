# [Domain] Shop 구현 계획

## 1. 개요
PVP 승리 보상으로 획득한 재화를 사용하여 성장에 필요한 아이템을 구매하는 순환 시스템입니다.

## 2. 주요 기능 (Features)

### 2.1 재화 관리
*   **투신의 증표 (PVP Coin)**:
    *   승리 시 획득 (티어별 차등 지급 가능).
    *   전용 재화 지갑(Wallet) 또는 `UserDataStorage` 속성으로 관리.

### 2.2 상점 이용
*   **상품 리스트**: `pvp_shop_items` 데이터셋 기반으로 판매 목록 로드.
*   **구매 프로세스**: 재화 보유량 확인 -> 재화 차감 -> 아이템(유닛/잡화) 지급.

## 3. 구현 파일
*   `Scripts/GameLogic/PVP/Shop/PVPShopService.mlua`


#!/bin/bash
#
# backup - 일정 기간이 지난 파일을 압축하여 백업 디렉터리로 이동하는 스크립트
#
# 사용법:
#   ./backup                   -> 현재 디렉터리에서 30일 지난 파일 백업
#   ./backup ~/Documents       -> Documents 디렉터리에서 30일 지난 파일 백업
#   ./backup ~/Downloads 60    -> Downloads 디렉터리에서 60일 지난 파일 백업
#

# ===== 백업 디렉터리 (스크립트 내에서 지정) =====
BACKUP_DIR="$HOME/backup"

# ===== 사용자 입력 인자 처리 =====
TARGET_DIR="${1:-.}"      # 첫 번째 인자가 없으면 현재 디렉터리
DAYS="${2:-30}"           # 두 번째 인자가 없으면 30일

# ===== 입력 검증 =====
# 대상 디렉터리 존재 여부 확인
if [ ! -d "$TARGET_DIR" ]; then
    echo "오류: 대상 디렉터리 '$TARGET_DIR'가 존재하지 않습니다." >&2
    exit 1
fi

# 날짜가 양의 정수인지 확인
if ! [[ "$DAYS" =~ ^[0-9]+$ ]] || [ "$DAYS" -le 0 ]; then
    echo "오류: 날짜는 양의 정수여야 합니다. (입력값: $DAYS)" >&2
    exit 1
fi

# ===== 백업 디렉터리 생성 (없으면) =====
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
    echo "백업 디렉터리 생성: $BACKUP_DIR"
fi

# ===== 대상 디렉터리 절대 경로로 변환 =====
TARGET_DIR=$(cd "$TARGET_DIR" && pwd)

echo "=============================================="
echo "대상 디렉터리 : $TARGET_DIR"
echo "기준 기간     : ${DAYS}일 이상 지난 파일"
echo "백업 디렉터리 : $BACKUP_DIR"
echo "=============================================="

# ===== 조건에 맞는 파일 찾기 =====
# -type f : 일반 파일만
# -mtime +N : N일 이전에 수정된 파일
# 백업 디렉터리가 대상 디렉터리 내부에 있을 수 있으므로 제외
FILE_LIST=$(find "$TARGET_DIR" -type f -mtime +"$DAYS" ! -path "$BACKUP_DIR/*" 2>/dev/null)

# 찾은 파일이 없는 경우 종료
if [ -z "$FILE_LIST" ]; then
    echo "${DAYS}일이 지난 파일이 없습니다."
    exit 0
fi

# 찾은 파일 개수 및 목록 출력
FILE_COUNT=$(echo "$FILE_LIST" | wc -l)
echo "찾은 파일 개수: $FILE_COUNT"
echo "----------------------------------------------"
echo "$FILE_LIST"
echo "----------------------------------------------"

# ===== 압축 파일 이름 생성 (현재 날짜와 시간 기반) =====
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
ARCHIVE_NAME="backup_${TIMESTAMP}.tar.gz"
ARCHIVE_PATH="$BACKUP_DIR/$ARCHIVE_NAME"

# ===== 파일 압축 =====
# tar로 묶어 백업 디렉터리에 저장
# -C / 를 사용하여 절대경로 상의 파일 구조를 유지하여 압축
echo "압축 중: $ARCHIVE_PATH"
echo "$FILE_LIST" | tar -czf "$ARCHIVE_PATH" -T - 2>/dev/null

# 압축 성공 여부 확인
if [ $? -ne 0 ] || [ ! -f "$ARCHIVE_PATH" ]; then
    echo "오류: 압축에 실패했습니다." >&2
    exit 1
fi

echo "압축 완료: $ARCHIVE_PATH"

# ===== 원본 파일 삭제 =====
echo "원본 파일 삭제 중..."
DELETED_COUNT=0
while IFS= read -r file; do
    if [ -f "$file" ]; then
        rm -f "$file"
        if [ $? -eq 0 ]; then
            DELETED_COUNT=$((DELETED_COUNT + 1))
        fi
    fi
done <<< "$FILE_LIST"

echo "=============================================="
echo "백업 완료!"
echo "  - 압축 파일      : $ARCHIVE_PATH"
echo "  - 삭제된 파일 수 : $DELETED_COUNT / $FILE_COUNT"
echo "=============================================="

exit 0

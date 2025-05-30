import streamlit as st
import requests
from bs4 import BeautifulSoup
import json
import urllib.parse

# --- 성별, 나이별 키워드 데이터 (필요시 수정 가능) ---
male_keywords = [
    "스테이크", "삼겹살", "치킨", "갈비", "돈까스", "피자", "햄버거", "라면", "카레", "치즈",
    "감자탕", "떡볶이", "파스타", "짬뽕", "마라탕", "부대찌개", "감자", "계란찜", "된장찌개", "순두부찌개",
    "돈불고기", "족발", "닭볶음탕", "회", "참치", "해물파전", "불닭", "김치찌개", "김밥", "토마토파스타"
]
female_keywords = [
    "샐러드", "오트밀", "닭가슴살", "과일", "두부", "콩나물국", "잡곡밥", "도라지무침", "과일주스", "현미밥",
    "오믈렛", "연어샐러드", "닭가슴살샐러드", "아보카도", "청포묵무침", "연두부", "구운야채", "토마토", "버섯볶음", "가지구이",
    "미역국", "단호박죽", "오트밀죽", "방울토마토", "시금치나물", "고구마구이", "코코넛워터", "아몬드", "야채수프", "로제파스타"
]
age_keywords = {
    "20": [...],  # 위 코드의 20대 키워드
    "30": [...],  # 위 코드의 30대 키워드
    "40": [...],  # 위 코드의 40대 키워드
    "50": [...],  # 위 코드의 50대 키워드
    "60": [...],  # 위 코드의 60대 키워드
}
def get_age_keywords(age):
    try:
        age = int(age)
        decade = str((age // 10) * 10)
        return age_keywords.get(decade, [])
    except:
        return []

st.title("맞춤형 레시피 추천 시스템")

region = st.selectbox(
    "거주 지역(나라 이름)을 선택하세요",
    ["한국", "중국", "일본", "미국", "이탈리아", "프랑스", "동남아", "태국", "베트남", "인도", "그리스", "멕시코", "스페인", "터키", "러시아", "영국", "독일", "브라질", "세계요리"]
)
gender = st.radio("성별을 선택하세요", ["남성", "여성"])
age = st.text_input("나이를 입력하세요 (숫자만)", "")

st.info(f"{region} 요리만 추천합니다. 재료는 한국어로 3개 이상 입력하세요. (예: 감자, 당근, 닭고기)")

ingredient_input = st.text_input("재료를 입력하세요 (쉼표 또는 공백으로 구분)", "")
if ingredient_input:
    if ',' in ingredient_input:
        ingredients = [ing.strip() for ing in ingredient_input.split(',') if ing.strip()]
    else:
        ingredients = [ing.strip() for ing in ingredient_input.split() if ing.strip()]

    user_keywords = []
    if gender == "남성":
        user_keywords += [k for k in male_keywords if k not in ingredients]
    elif gender == "여성":
        user_keywords += [k for k in female_keywords if k not in ingredients]
    user_keywords += [k for k in get_age_keywords(age) if k not in ingredients and k not in user_keywords]
    final_ingredients = ingredients + user_keywords[:7]

    if len(ingredients) < 3:
        st.error("❗ 재료는 3개 이상 입력해야 합니다.")
    else:
        with st.spinner("레시피 검색 중..."):
            region_map = {
                "한국": "한식",
                "중국": "중식",
                "일본": "일식"
            }
            category = region_map.get(region, region)
            query = "+".join(final_ingredients)
            url = f"https://www.10000recipe.com/recipe/list.html?q={category}&order=reco"
            res = requests.get(url)
            soup = BeautifulSoup(res.text, 'html.parser')
            recipes = soup.select("ul.common_sp_list_ul.ea4 li")[:20]

            results = []
            for item in recipes:
                title_tag = item.select_one(".common_sp_caption_tit")
                link_tag = item.select_one("a")
                if not title_tag or not link_tag:
                    continue
                title = title_tag.get_text(strip=True)
                link = "https://www.10000recipe.com" + link_tag["href"]
                # 상세 페이지에서 재료 추출
                detail = requests.get(link)
                detail_soup = BeautifulSoup(detail.text, 'html.parser')
                script_tag = detail_soup.find("script", {"type": "application/ld+json"})
                if not script_tag:
                    continue
                try:
                    json_data = json.loads(script_tag.string)
                    recipe_ingredients = json_data.get("recipeIngredient", [])
                except:
                    continue
                matched = [i for i in ingredients if any(i in ing for ing in recipe_ingredients)]
                unmatched = [i for i in recipe_ingredients if not any(ing in i for ing in ingredients)]
                match_rate = round(len(matched) / len(recipe_ingredients) * 100, 1) if recipe_ingredients else 0
                results.append({
                    "title": title,
                    "link": link,
                    "match_rate": match_rate,
                    "matched": matched,
                    "unmatched": unmatched
                })

            results = sorted(results, key=lambda x: -x['match_rate'])[:5]

        if not results:
            st.warning("😢 입력한 재료와 일치하는 레시피가 없습니다.")
        else:
            st.subheader(f"🍽️ [{region}] 맞춤형 레시피 추천 결과 (Top 5):")
            for i, r in enumerate(results, 1):
                st.markdown(f"**{i}. [{r['title']}]({r['link']})** ({r['match_rate']}%)")
                st.markdown(f"✅ **일치한 재료:** {', '.join(r['matched']) if r['matched'] else '없음'}")
                st.markdown(f"❌ **부족한 재료:** {', '.join(r['unmatched']) if r['unmatched'] else '없음'}")
                if r['unmatched']:
                    st.markdown("🛒 **장바구니**")
                    for item in r['unmatched']:
                        encoded = urllib.parse.quote_plus(item)
                        link = f"https://www.coupang.com/np/search?component=&q={encoded}"
                        st.markdown(f"- [{item}]({link})")
                st.write("")

# age_keywords 값 채우기 예시 (생략된 부분을 위 코드에서 복사해서 넣으세요)

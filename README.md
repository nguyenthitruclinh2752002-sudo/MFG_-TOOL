# MFG_-TOOL
AUTO READ DATECODE
import datetime
import pandas as pd
import streamlit as st

if "rules" not in st.session_state:
  st.session_state.rules = [
      {
          "mfg_dc": "A2026",
          "buyer": "Buyer X",
          "decoded_result": "Lô hàng tháng 10/2026 - Kho chính",
          "updated_by": "Admin (Lina)",
          "updated_at": "2026-10-13 10:00",
      }
  ]

if "history" not in st.session_state:
  st.session_state.history = []

if "users" not in st.session_state:
  st.session_state.users = {"Lina": "Admin", "IT_Reviewer": "Viewer"}

st.set_page_config(page_title="MFG/DC Auto Decode Tool (MVP)", layout="wide")

st.title("📦 Hệ Thống Tra Cứu & Quản Lý Quy Tắc MFG/DC")

current_user = st.sidebar.selectbox(
    "Chọn tài khoản đăng nhập:", list(st.session_state.users.keys())
)
current_role = st.session_state.users[current_user]
st.sidebar.info(f"Vai trò hiện tại: **{current_role}**")

menu = st.sidebar.radio(
    "Chức năng:",
    [
        "🔍 Auto Decode",
        "⚙️ Rule Management",
        "📥 Import Excel",
        "👥 User Management",
        "📜 Verification & History",
    ],
)

if menu == "🔍 Auto Decode":
  st.header("Tra Cứu Quy Tắc MFG / DC")
  search_code = st.text_input("Nhập mã MFG hoặc DC cần tra cứu:").strip()
  if search_code:
    found = [
        r
        for r in st.session_state.rules
        if search_code.upper() in r["mfg_dc"].upper()
    ]
    if found:
      st.success(f"Tìm thấy {len(found)} kết quả phù hợp:")
      st.table(
          pd.DataFrame(found)[["mfg_dc", "buyer", "decoded_result", "updated_at"]]
      )
    else:
      st.warning("Không tìm thấy quy tắc cho mã này.")

elif menu == "⚙️ Rule Management":
  st.header("Quản Lý Quy Tắc")
  if current_role != "Admin":
    st.error("Bạn không có quyền truy cập chức năng này.")
  else:
    with st.form("add_rule_form"):
      new_mfg = st.text_input("Mã MFG/DC").strip().upper()
      new_buyer = st.text_input("Tên Buyer").strip()
      new_result = st.text_area("Kết quả giải mã").strip()
      submit = st.form_submit_button("Lưu quy tắc")
      if submit and new_mfg and new_result:
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M")
        st.session_state.rules.append({
            "mfg_dc": new_mfg,
            "buyer": new_buyer,
            "decoded_result": new_result,
            "updated_by": current_user,
            "updated_at": timestamp,
        })
        st.success(f"Đã lưu thành công quy tắc cho mã: {new_mfg}")
    st.dataframe(pd.DataFrame(st.session_state.rules))

elif menu == "📥 Import Excel":
  st.header("Nhập dữ liệu từ Excel")
  if current_role != "Admin":
    st.error("Bạn không có quyền thực hiện tính năng này.")
  else:
    uploaded_file = st.file_uploader("Chọn file Excel", type=["xlsx", "xls"])
    if uploaded_file:
      df_import = pd.read_excel(uploaded_file)
st.write("Xem trước dữ liệu:", df_import.head())

elif menu == "👥 User Management":
  st.header("Quản Lý Người Dùng")
  st.table(
      pd.DataFrame(
          list(st.session_state.users.items()), columns=["Username", "Role"]
      )
  )

elif menu == "📜 Verification & History":
  st.header("Lịch Sử Chỉnh Sửa")
  if st.session_state.history:
    st.dataframe(pd.DataFrame(st.session_state.history))
  else:
    st.info("Chưa có lịch sử thay đổi.")

# pbtype
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm
import matplotlib
import os

def get_valid_file_path():
    while True:
        csv_file = input("\nファイルのパスを入力してください (終了する場合は 'exit' と入力): ").strip()
        
        if csv_file.lower() == 'exit':
            print("プログラムを終了します。")
            exit()
        
        # 先頭と末尾のダブルクォーテーションやシングルクォーテーションを削除
        if (csv_file.startswith('"') and csv_file.endswith('"')) or (csv_file.startswith("'") and csv_file.endswith("'")):
            csv_file = csv_file[1:-1]
        
        # バックスラッシュをスラッシュに変換
        csv_file = csv_file.replace("\\", "/")
        
        # ファイルの存在を確認
        if os.path.isfile(csv_file):
            return csv_file
        else:
            print("指定されたファイルが見つかりません。もう一度入力してください。")

def main():
    # ファイルパスの入力を取得
    csv_file = get_valid_file_path()
    
    # CSVファイルの読み込み
    try:
        df = pd.read_csv(csv_file, encoding='utf-8')
    except Exception as e:
        print(f"CSVファイルの読み込み中にエラーが発生しました: {e}")
        return
    
    # 投手名の入力を取得
    pitcher_name = input("投手名を入力してください: ").strip()
    
    # 指定した投手名でデータをフィルタリング
    filtered_df = df[df['pitcher_name'] == pitcher_name]
    
    if filtered_df.empty:
        print(f"投手名 '{pitcher_name}' のデータが見つかりません。プログラムを終了します。")
        return
    
    # pitched_ball_type ごとの pitched_result の内訳を集計
    result_summary = filtered_df.groupby(['pitched_ball_type', 'pitched_result']).size().unstack(fill_value=0)
    
    # 割合に変換
    result_percentage = result_summary.div(result_summary.sum(axis=1), axis=0) * 100
    
    # フォントのパスを指定
    font_path = r'C:\Users\santokentaro\Downloads\Noto_Sans_JP\NotoSansJP-VariableFont_wght.ttf'
    
    if not os.path.isfile(font_path):
        print(f"指定されたフォントファイルが見つかりません: {font_path}")
        return
    
    # フォントを登録
    fm.fontManager.addfont(font_path)
    font_prop = fm.FontProperties(fname=font_path)
    
    # グローバルにフォントを設定
    plt.rcParams['font.family'] = 'Noto Sans JP'
    plt.rcParams['font.sans-serif'] = ['Noto Sans JP']
    plt.rcParams['axes.unicode_minus'] = False  # マイナス記号が文字化けしないように設定
        
    # 可視化（スタックバー）と割合を数字で表示
    fig, ax = plt.subplots(figsize=(12, 8))
    result_percentage.plot(kind='bar', stacked=True, colormap='tab20', ax=ax)
    
    # バーの上に割合を表示
    for container in ax.containers:
        ax.bar_label(
            container,
            fmt='%.1f%%',
            label_type='center',
            fontsize=10,
            fontproperties=font_prop,
            color='white'  # 文字色を白に固定
        )
    
    # タイトルとラベルの設定
    plt.title(f'{pitcher_name} の Pitched Ball Type ごとの Pitched Result の内訳（%）', fontsize=16, fontproperties=font_prop)
    plt.xlabel('Pitched Ball Type', fontsize=14, fontproperties=font_prop)
    plt.ylabel('割合 (%)', fontsize=14, fontproperties=font_prop)
    plt.xticks(rotation=45, fontsize=12, fontproperties=font_prop)
    plt.yticks(fontsize=12, fontproperties=font_prop)
    
    # 凡例の設定
    plt.legend(title='Pitched Result', prop=font_prop)
    
    # プロットの表示
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    main()

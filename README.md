# Aleka
Интуитивный инструмент для вейвлет-анализа временных рядов  Добро пожаловать в Aleka — ваш помощник в визуализации и анализе данных! Вы можете загружать CSV и Excel файлы, проводить вейвлет-анализ, строить графики и сохранять результаты.
# 1. Импорт библиотек 1
!pip install PyWavelets seaborn
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import pywt
from statsmodels.tsa.seasonal import STL
from google.colab import files

# 2. Загрузка файла
print("⬇ Пожалуйста, выбери CSV или Excel файл:")
uploaded = files.upload()

# 3. Определение имени файла
import io
import os

filename = list(uploaded.keys())[0]
file_ext = os.path.splitext(filename)[1].lower()

# 4. Чтение файла (CSV или Excel)
if file_ext == '.csv':
    df = pd.read_csv(io.StringIO(uploaded[filename].decode('utf-8')))
elif file_ext in ['.xls', '.xlsx']:
    df = pd.read_excel(io.BytesIO(uploaded[filename]))
else:
    raise ValueError('⚠️ Поддерживаются только форматы CSV и Excel (.xls, .xlsx)')

# 5. Проверка структуры и очистка
print("📄 Первые строки таблицы:")
display(df.head())

# Подстройка под названия колонок
df.columns = [col.strip().capitalize() for col in df.columns]  # Упрощаем названия

if 'Date' not in df.columns or 'Value' not in df.columns:
    raise ValueError("⛔ Убедись, что в таблице есть столбцы 'Date' и 'Value'")

df['Date'] = pd.to_datetime(df['Date'])
df = df[['Date', 'Value']].dropna().sort_values('Date')

# 6. STL-декомпозиция
df_stl = df.set_index('Date')
stl = STL(df_stl['Value'], period=12, robust=True)
result = stl.fit()
df_stl['Trend'] = result.trend
df_stl['Seasonal'] = result.seasonal
df_stl['Residual'] = result.resid

# 7. График STL
result.plot()
plt.suptitle('STL-декомпозиция временного ряда', fontsize=14)
plt.tight_layout()
plt.show()
# 📦 Установка виджетов (если ещё не установлены)
!pip install ipywidgets --quiet

# 📥 Импорт виджетов
import ipywidgets as widgets
from IPython.display import display, clear_output

#  Список доступных вейвлетов
wavelet_options = ['morl', 'cmor', 'mexh', 'gaus1', 'gaus4', 'shan', 'fbsp']

# 📍 Виджет выбора
wavelet_dropdown = widgets.Dropdown(
    options=wavelet_options,
    value='morl',
    description='Вейвлет:',
    style={'description_width': 'initial'},
    layout=widgets.Layout(width='300px')
)

# 👇 Кнопка для запуска анализа
run_button = widgets.Button(description="Запустить вейвлет-анализ", button_style='success')

# 🔁 Callback-функция
def run_wavelet_analysis(b):
    clear_output(wait=True)
    display(wavelet_dropdown, run_button)

    selected_wavelet = wavelet_dropdown.value
    print(f"📈 Используем вейвлет: {selected_wavelet}")

    # ⏬ Выполняем CWT
    signal_clean = df_stl['Residual'].dropna()
    dates_clean = signal_clean.index
    values_clean = signal_clean.values

    scales = np.arange(1, 120)
    coef, freqs = pywt.cwt(values_clean, scales, selected_wavelet, sampling_period=1)
    periods = 1 / freqs

    # 🔥 Визуализация
    plt.figure(figsize=(16, 6))
    ax = sns.heatmap(np.abs(coef), cmap='viridis', cbar_kws={'label': 'Амплитуда'},
                     xticklabels=30, yticklabels=10)

    ax.set_xticks(np.linspace(0, len(dates_clean)-1, 10))
    ax.set_xticklabels([dates_clean[int(i)].strftime('%Y-%m') for i in np.linspace(0, len(dates_clean)-1, 10)],
                       rotation=90)

    ax.set_yticks(np.linspace(0, len(periods)-1, 8))
    ax.set_yticklabels([f'{periods[int(i)]:.1f}' for i in np.linspace(0, len(periods)-1, 8)],rotation=0)

    ax.set_xlabel('Дата')
    ax.set_ylabel('Период (год)')
    plt.title(f'Вейвлет-анализ (вейвлет: {selected_wavelet})')
    plt.tight_layout()
    plt.show()

# 🔘 Привязка кнопки к функции
run_button.on_click(run_wavelet_analysis)

# 🚀 Отображаем интерфейс
display(wavelet_dropdown, run_button)

import ipywidgets as widgets
from IPython.display import display, clear_output
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import cm

#  Выбор вейвлета
wavelet_dropdown = widgets.Dropdown(
    options=['morl', 'cmor', 'mexh', 'gaus1','gaus4', 'shan', 'fbsp'],
    value='morl',
    description='Вейвлет:',
    layout=widgets.Layout(width='250px')
)

# 🔄 Переключатель 2D / 3D
mode_toggle = widgets.ToggleButtons(
    options=['2D', '3D'],
    description='График:',
    button_style='info'
)

# ▶️ Кнопка запуска
run_button = widgets.Button(description="Построить", button_style='success')

# 📈 Функция запуска
def run_wavelet_analysis(_):
    clear_output(wait=True)
    display(wavelet_dropdown, mode_toggle, run_button)

    wavelet = wavelet_dropdown.value
    mode = mode_toggle.value

    print(f"Вейвлет: {wavelet} | Режим: {mode}")

    signal_clean = df_stl['Residual'].dropna()
    dates_clean = signal_clean.index
    values_clean = signal_clean.values

    scales = np.arange(1, 132)
    coef, freqs = pywt.cwt(values_clean, scales, wavelet, sampling_period=1)
    periods = 1 / freqs

    if mode == '2D':
        plt.figure(figsize=(16, 6))
        ax = sns.heatmap(np.abs(coef), cmap='viridis', cbar_kws={'label': 'Амплитуда'},
                         xticklabels=30, yticklabels=10)

        ax.set_xticks(np.linspace(0, len(dates_clean)-1, 47))
        ax.set_xticklabels([dates_clean[int(i)].strftime('%Y') for i in np.linspace(0, len(dates_clean)-1, 47)],
                           rotation=90)
        ax.set_yticks(np.linspace(0, len(periods)-1, 16))
        ax.set_yticklabels([f'{periods[int(i)]:.1f}' for i in np.linspace(0, len(periods)-1, 16)], rotation=0)

        ax.set_xlabel('Дата')
        ax.set_ylabel('Период (месяц)')
        plt.title(f'2D Вейвлет-анализ ({wavelet})')
        plt.tight_layout()
        plt.show()

    elif mode == '3D':
        X = np.arange(len(dates_clean))
        Y = periods
        X, Y = np.meshgrid(X, Y)
        Z = np.abs(coef)

        fig = plt.figure(figsize=(18, 9))
        ax = fig.add_subplot(111, projection='3d')
        surf = ax.plot_surface(X, Y, Z, cmap=cm.viridis, edgecolor='none')

        ax.set_xlabel('Дата')
        ax.set_ylabel('Период (месяц)')
        ax.set_zlabel('Амплитуда')

        xticks_locs = np.linspace(0, len(dates_clean) - 1, 10).astype(int)
        ax.set_xticks(xticks_locs)
        ax.set_xticklabels([dates_clean[i].strftime('%Y-%m') for i in xticks_locs], rotation=45, ha='right')

        fig.colorbar(surf, shrink=0.5, aspect=12, label='Амплитуда')
        plt.title(f'🌊 3D Вейвлет-анализ ({wavelet})')
        plt.tight_layout()
        plt.show()

# 🎛️ Связь кнопки с функцией
run_button.on_click(run_wavelet_analysis)

# 🚀 Отображение интерфейса
display(wavelet_dropdown, mode_toggle, run_button)

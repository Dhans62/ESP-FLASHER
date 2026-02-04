#!/bin/bash

# --- WARNA & UI ---
GREEN='\033[0;32m'
BLUE='\033[0;34m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
RED='\033[0;31m'
NC='\033[0m'

# --- KONFIGURASI PATH ---
ARDUINO_DIR="$HOME/Arduino"
BIN_DIR="$ARDUINO_DIR/bin"
CLI_PATH="$BIN_DIR/arduino-cli"
DATA_DIR="$HOME/.arduino15"

# --- FUNGSI ALTERNATIVE SCREEN ---
open_alt_screen() { echo -ne "\033[?1049h\033[H"; }
close_alt_screen() { echo -ne "\033[?1049l"; }

trap 'close_alt_screen; exit' SIGINT SIGTERM

display_header() {
    clear
    echo -e "${CYAN}  ______  _____  _____    ______ _                _      ${NC}"
    echo -e "${CYAN} |  ____|/ ____||  __ \  |  ____| |              | |     ${NC}"
    echo -e "${CYAN} | |__  | (___  | |__) | | |__  | |  __ _  ___| |__  ${NC}"
    echo -e "${CYAN} |  __|  \___ \ |  ___/  |  __| | | / _\` |/ __| '_ \ ${NC}"
    echo -e "${CYAN} | |____ ____) || |      | |    | |  (_| |\__ \ | | |${NC}"
    echo -e "${CYAN} |______|_____/ |_|      |_|    |_|\__,_||___/_| |_|${NC}"
    echo -e "${YELLOW}  By Dhann | Credits: Arduino IDE${NC}"
    echo -e "${BLUE}-----------------------------------------------------${NC}"
}

check_engine() {
    if [ ! -f "$CLI_PATH" ]; then
        echo -e "${YELLOW} [!] Engine tidak ditemukan. Mengunduh...${NC}"
        mkdir -p "$BIN_DIR"
        curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=$BIN_DIR sh
        $CLI_PATH config init --overwrite > /dev/null
        $CLI_PATH config set directories.data "$DATA_DIR"
        $CLI_PATH config set directories.user "$ARDUINO_DIR"
        $CLI_PATH core update-index
    fi
}

# --- TAHAP LIBRARY MANAGER ---
manage_libraries() {
    while true; do
        display_header
        echo -e "${GREEN}--- LIBRARY MANAGER ---${NC}"
        echo -e " [1] Cari & Install Library"
        echo -e " [2] List Library Terpasang"
        echo -e " [3] Update Database (Real-time)"
        echo -e " [4] Kembali"
        echo -e "${BLUE}-----------------------------------------------------${NC}"
        read -e -p " Pilih Aksi: " lib_act

        case $lib_act in
            1)
                read -e -p " Masukkan nama/keyword library: " kw
                [[ "$kw" =~ [[:cntrl:]] ]] && kw=""
                [ -z "$kw" ] && continue
                
                echo -e "${YELLOW} [..] Mencari di Database...${NC}"
                mapfile -t results < <($CLI_PATH lib search "$kw" --format text | grep "Name:" | sed 's/Name: //g' | head -n 15)
                mapfile -t versions < <($CLI_PATH lib search "$kw" --format text | grep "Latest:" | sed 's/Latest: //g' | head -n 15)

                if [ ${#results[@]} -eq 0 ]; then
                    echo -e "${RED} [!] Library tidak ditemukan.${NC}"
                    sleep 1; continue
                fi

                echo -e "\n${CYAN}HASIL PENCARIAN:${NC}"
                for i in "${!results[@]}"; do
                    echo -e " [${GREEN}$((i+1))${NC}] ${results[$i]} ${YELLOW}(v${versions[$i]})${NC}"
                done
                echo -e "${BLUE}-----------------------------------------------------${NC}"
                
                read -e -p " Pilih nomor library (Kosongkan untuk batal): " lib_num
                if [[ "$lib_num" =~ ^[0-9]+$ ]] && [ "$lib_num" -le "${#results[@]}" ]; then
                    selected_lib="${results[$((lib_num-1))]}"
                    
                    while true; do
                        display_header
                        echo -e "${GREEN}--- DETAIL: $selected_lib ---${NC}"
                        $CLI_PATH lib details "$selected_lib" | grep -E "Summary:|Website:" || echo "No extra info found."
                        echo -e "${BLUE}-----------------------------------------------------${NC}"
                        echo -e " [1] Install Versi Terbaru (${versions[$((lib_num-1))]})"
                        echo -e " [2] Pilih Versi Spesifik"
                        echo -e " [3] Batal"
                        read -e -p " Pilih Aksi: " det_act
                        
                        case $det_act in
                            1)
                                echo -e "${YELLOW} [..] Menginstal $selected_lib...${NC}"
                                $CLI_PATH lib install "$selected_lib"
                                read -p " Selesai. Tekan ENTER..."; break 2 ;;
                            2)
                                echo -e "${YELLOW} [..] Mengambil daftar versi...${NC}"
                                $CLI_PATH lib search "$selected_lib" --format text | grep "Versions:" | sed 's/Versions: //'
                                read -e -p " Ketik versi yang diinginkan (ex: 1.0.2): " spec_ver
                                if [ -n "$spec_ver" ]; then
                                    $CLI_PATH lib install "$selected_lib@$spec_ver"
                                    read -p " Selesai. Tekan ENTER..."; break 2
                                fi ;;
                            3) break ;;
                        esac
                    done
                fi ;;
            2)
                echo -e "${GREEN} [i] Library yang sudah terinstal:${NC}"
                $CLI_PATH lib list
                read -p " Tekan ENTER..." ;;
            3)
                echo -e "${YELLOW} [..] Sinkronisasi Database Real-time...${NC}"
                $CLI_PATH lib update-index
                read -p " Database diperbarui. Tekan ENTER..." ;;
            4) break ;;
        esac
    done
}

# --- TAHAP 3: ADVANCED BOARD CONFIG ---
setup_new_config() {
    display_header
    echo -e "${GREEN}--- ADVANCED BOARD SETUP ---${NC}"
    echo -e " 1) ESP32-C3 Dev Module | 2) ESP32 Dev Module | 3) Other"
    read -e -p " Board Type: " b_choice
    case $b_choice in
        1) FQBN_BASE="esp32:esp32:esp32c3" ;;
        2) FQBN_BASE="esp32:esp32:esp32" ;;
        *) read -e -p " Manual FQBN: " FQBN_BASE ;;
    esac

    echo -e "\n${YELLOW}[ADVANCED EXPLANATION]${NC}"
    echo -e " ${CYAN}1. USB CDC On Boot${NC} : Jalur Serial USB internal (Penting untuk C3/S2/S3)."
    echo -e " ${CYAN}2. CPU Frequency${NC}   : 160MHz (Performance) vs 80MHz (Power Saving)."
    echo -e " ${CYAN}3. Flash Mode${NC}      : QIO (Cepat) vs DIO (Kompatibel)."
    echo -e " ${CYAN}4. Partition${NC}       : Huge APP memberi ruang kode hingga 3MB."
    echo -e " ${CYAN}5. Erase Flash${NC}     : Membersihkan sisa data lama di memori chip."
    echo -e "${BLUE}-----------------------------------------------------${NC}"

    # --- LOGIKA CERDAS CDC ---
    CDC_PARAM=""
    if [[ "$FQBN_BASE" == *"c3"* ]] || [[ "$FQBN_BASE" == *"s2"* ]] || [[ "$FQBN_BASE" == *"s3"* ]]; then
        read -e -p " USB CDC On Boot [1: Enabled, 2: Disabled]: " cdc_choice
        [ "$cdc_choice" == "1" ] && CDC_VAL="cdc" || CDC_VAL="default"
        CDC_PARAM="CDCOnBoot=${CDC_VAL},"
    fi

    read -e -p " CPU Frequency   [1: 160MHz, 2: 80MHz]: " cpu_choice
    [ "$cpu_choice" == "2" ] && CPU="80" || CPU="160"
    read -e -p " Upload Speed    [1: 921600, 2: 115200]: " s_choice
    [ "$s_choice" == "1" ] && BAUD="921600" || BAUD="115200"
    read -e -p " Flash Freq      [1: 80MHz, 2: 40MHz]: " f_choice
    [ "$f_choice" == "1" ] && F_FREQ="80" || F_FREQ="40"
    read -e -p " Flash Mode      [1: QIO, 2: DIO]: " m_choice
    [ "$m_choice" == "1" ] && F_MODE="qio" || F_MODE="dio"
    read -e -p " Flash Size      [1: 4MB, 2: 2MB]: " sz_choice
    [ "$sz_choice" == "1" ] && F_SIZE="4M" || F_SIZE="2M"
    read -e -p " Partition       [1: Default, 2: Huge APP]: " p_choice
    [ "$p_choice" == "2" ] && PART="huge_app" || PART="default"
    read -e -p " Debug Level     [1: None, 2: Info]: " d_choice
    [ "$d_choice" == "2" ] && DBG="info" || DBG="none"
    read -e -p " Erase All Flash [1: No, 2: YES]: " e_choice
    [ "$e_choice" == "2" ] && ERASE="true" || ERASE="false"

    FQBN="${FQBN_BASE}:${CDC_PARAM}CPUFreq=${CPU},FlashFreq=${F_FREQ},FlashMode=${F_MODE},FlashSize=${F_SIZE},PartitionScheme=${PART},DebugLevel=${DBG}"
    
    echo "FQBN=\"$FQBN\"" > "$CONFIG_FILE"
    echo "BAUD=\"$BAUD\"" >> "$CONFIG_FILE"
    echo "ERASE=\"$ERASE\"" >> "$CONFIG_FILE"
    source "$CONFIG_FILE"
}

# --- MAIN SCRIPT ---
open_alt_screen
check_engine

display_header
echo -e "${GREEN}--- PILIH PROYEK ---${NC}"
cd "$ARDUINO_DIR" || { close_alt_screen; exit 1; }
projects=($(ls -d */ | grep -vE 'bin/|libraries/' | sed 's/\///'))
for i in "${!projects[@]}"; do echo -e " [${CYAN}$((i+1))${NC}] ${projects[$i]}"; done
read -e -p " Select project: " p_num
selected_project="${projects[$((p_num-1))]}"
[ -z "$selected_project" ] && { close_alt_screen; exit 1; }

PROJECT_PATH="$ARDUINO_DIR/$selected_project"
CONFIG_FILE="$PROJECT_PATH/.esp_config"

if [ -f "$CONFIG_FILE" ]; then
    source "$CONFIG_FILE"
    display_header
    echo -e "${YELLOW} [i] Konfigurasi Aktif:${NC}"
    echo -e " Board      : ${CYAN}$FQBN${NC}"
    echo -e " Wipe Flash : ${CYAN}$ERASE${NC}"
    echo -e "${BLUE}-----------------------------------------------------${NC}"
    read -e -p " Change Config? (y/N): " chg
    [[ "$chg" =~ ^[Yy]$ ]] && setup_new_config
else
    setup_new_config
fi

while true; do
    display_header
    echo -e " ${GREEN}PROYEK --> ${NC}${YELLOW}$selected_project${NC}"
    echo -e "${BLUE}-----------------------------------------------------${NC}"
    echo -e " [1] View Files      [2] Edit Code"
    echo -e " [3] Compile & Flash [4] Serial Monitor"
    echo -e " [5] Manage Library  [6] Exit / Switch"
    echo -e "${BLUE}-----------------------------------------------------${NC}"
    read -e -p " Masukkan Aksi: " action

    case $action in
        1)
            echo -e "\n${GREEN}--- DAFTAR FILE ---${NC}"
            ls -F --color=auto "$PROJECT_PATH"
            read -p " Tekan ENTER..." ;;
        2)
            files=($(ls "$PROJECT_PATH" | grep -v ".esp_config"))
            for i in "${!files[@]}"; do echo -e " [${CYAN}$((i+1))${NC}] ${files[$i]}"; done
            read -e -p " Pilih nomor file: " f_choice
            target_file="${files[$((f_choice-1))]}"
            [ -n "$target_file" ] && nano "$PROJECT_PATH/$target_file" ;;
        3)
            echo -e "${YELLOW} [..] Mengompilasi untuk $FQBN...${NC}"
            if $CLI_PATH compile --fqbn "$FQBN" "$PROJECT_PATH"; then
                echo -e "${YELLOW} [..] Mencari Port (Menunggu OTG dicolok)...${NC}"
                PORT=""
                for i in {1..10}; do
                    PORT=$(ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null | head -n 1)
                    if [ -n "$PORT" ]; then break; fi
                    echo -ne "${RED} [!] Port belum ada, mencoba lagi ($i/10)... \r${NC}"
                    sleep 1
                done
                echo ""

                if [ -z "$PORT" ]; then
                    echo -e "${RED} [!] Port Tetap Tidak Ditemukan.${NC}"
                    echo -e "${CYAN} [i] Memeriksa Hardware via lsusb:${NC}"
                    lsusb
                    echo -e "${BLUE}-----------------------------------------------------${NC}"
                    echo -e " [1] Masukkan Path Port Manual"
                    echo -e " [2] Batal"
                    read -e -p " Pilih Tindakan: " u_fail
                    if [ "$u_fail" == "1" ]; then
                        read -e -p " Masukkan Port (ex: /dev/ttyUSB0): " PORT
                    fi
                fi

                if [ -n "$PORT" ]; then
                    sudo chmod 666 "$PORT" 2>/dev/null || echo "Info: Gagal chmod, mencoba upload..."
                    EXTRA_FLAGS=""
                    [ "$ERASE" == "true" ] && EXTRA_FLAGS="--upload-field erase-all=true"
                    $CLI_PATH upload -p "$PORT" --fqbn "$FQBN" --upload-speed "$BAUD" $EXTRA_FLAGS "$PROJECT_PATH"
                else
                    echo -e "${RED} [!] Upload Dibatalkan.${NC}"
                fi
            fi
            read -p " Tekan ENTER..." ;;
        4)
            PORT=$(ls /dev/ttyACM* /dev/ttyUSB* 2>/dev/null | head -n 1)
            [ -n "$PORT" ] && $CLI_PATH monitor -p "$PORT" -c baudrate=115200 || echo -e "${RED} [!] Tidak Ada Port.${NC}"
            read -p " Tekan ENTER..." ;;
        5) manage_libraries ;;
        6) close_alt_screen; exit 0 ;;
    esac
done

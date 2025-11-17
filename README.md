# MyDemo


react-native bundle
--dev false
--entry-file index.ts
--bundle-output ios/main.jsbundle
--platform ios
--assets-dest ios




https://teams.microsoft.com/l/message/19:62d47693-c760-44a9-9f23-392626d30448_c03061d4-00c7-4421-b3af-cb6754b85ea0@unq.gbl.spaces/1761813668398?context=%7B%22contextType%22%3A%22chat%22%7D




                        onClose={() => setModalVisible(false)}
                        onProceed={() => setModalVisible(false)}
                    />
                </ScreenContainer>
                <InfoBar />
            </Fragment>
        </>
    );
};

const styles = StyleSheet.create({
    completeMessageContainer: {
        width: "100%",
        padding: 35,
        alignItems: "center",
        borderRadius: 16,
    },
    buttonsContainer: {
        flexDirection: "row",
        width: "100%",
        justifyContent: "space-between",
    },
    headerContainer: {
        flexDirection: "row",
        alignItems: "center",
        justifyContent: "space-between",
        paddingHorizontal: 16,
        paddingVertical: 22,
        backgroundColor: "#fff",
        elevation: 2, // ✅ Adds shadow on Android
        shadowColor: "#000", // ✅ Adds shadow on iOS
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 3,
        borderBottomWidth: 1,
        borderColor: "#eee",
    },
    headerText: {
        fontSize: 16,
        fontWeight: "bold",
        flex: 1,
        textAlign: "center"
    },
});


export async function calculateAvailableStorage() {
    const osReserved = 20000; // Reserved for OS
    const otherAppReserved = 5000; // Reserved for other apps
    const [, storage] = await Vitals.getStorage();
    const totalAvailableStorage = Number(storage?.total) - (osReserved + otherAppReserved);
    const actualAvailableStorage = totalAvailableStorage / 4; // To allow for backup and export
    return actualAvailableStorage / 1024;
}

    const exportExcelFile = useCallback(async (
        sourceFilePath: string,   // the Excel file you generated with exportDataToExcel
        outputPath: string,       // where the user wants to save it
    ): Promise<ErrorType> => {
        try {
            // Ensure the uri is in the correct format
            const [platformErr, uri] = Platform.select<string>({
                android: outputPath,
                ios: decodeURIComponent(outputPath)?.replace?.("file://", ""),
            });

            if (platformErr || !uri) {
                return error.report(MODULE_ID, "Error reformatting uri.", false);
            }

            // Copy the Excel file to the desired location
            try {
                await FS.copyFile(sourceFilePath, uri);
                console.log("Excel file exported to:", uri);
            } catch (copyErr) {
                return error.report(MODULE_ID, "Error copying Excel file.", false, copyErr);
            }

        } catch (e) {
            return error.report(MODULE_ID, "Error occurred during Excel export.", false, e);
        }
        return null;
    }, [error]);

   const getDateTimeString = useCallback((): DataWithErrorType<string> => {
        const utcMsSinceEpoch = getLocalMsSinceEpoch();
        const [err, dateTimeString] = epochToDateTimeString(utcMsSinceEpoch / 1000);
        if (!err && dateTimeString !== null) {
            return [null, dateTimeString];
        }
        return [error.report(MODULE_ID, "Invalid Date Time String", false, err), null];
    }, [epochToDateTimeString, error, getLocalMsSinceEpoch]);

export default MOSkinCalibrationNewCompleteScreen;


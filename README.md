# Hotkeys

## Add Application Settings like this, 5 values for each hotkey
    RegisterHotkeys		bool 	User	True
    hkMyHotkeyKey		string	User	PrintScreen
    hkMyHotkeyCtrl		bool	User	false
    hkMyHotkeyAlt		bool	User	false
    hkMyHotkeyShift		bool	User	false
    hkMyHotkeyWin		bool	User	false

## In HotkeyTools.cs, add a using statement to be able to reference Settings.Default, example:
    using MyApp.Properties;

## Add these methods to your main form, adjust HandleHotkey to taste

    Settings settings = Settings.Default;

    // For each hotkey below, add entries in Settings, hk???Key, hk???Ctrl, hk???Alt, hk???Shift, hk???Win
    public List<string> HotkeyNames = new List<string>
    {
            "MyHotkey1",
            "MyHotkey2",
    };
    public Dictionary<string, Hotkey> HotkeyList = new Dictionary<string, Hotkey>();
    
    public MainForm()
    {
            InitializeComponent();
    
            HotkeyList = HotkeyTools.LoadHotkeys(HotkeyList, HotkeyNames, this);
            if (settings.RegisterHotkeys) // optional
            {
                    HotkeyTools.RegisterHotkeys(HotkeyList);
                    // RegisterHotkeys returns a string[] with any failed hotkey registrations that can be used to output an error message
            }
    }
    
    private void Form1_FormClosing(object sender, FormClosingEventArgs e)
    {
            HotkeyTools.ReleaseHotkeys(HotkeyList);
    }
    
    protected override void WndProc(ref Message m)
    {
            base.WndProc(ref m);
            if (m.Msg == Hotkeys.Constants.WM_HOTKEY_MSG_ID)
            {
                    Keys key = (Keys)(((int)m.LParam >> 16) & 0xFFFF);                  // The key of the hotkey that was pressed.
                    KeyModifier modifier = (KeyModifier)((int)m.LParam & 0xFFFF);       // The modifier of the hotkey that was pressed.
                    int id = m.WParam.ToInt32();                                        // The id of the hotkey that was pressed.
                    //MessageBox.Show("Hotkey " + id + " has been pressed!");
                    HandleHotkey(id);
            }
    }
    
    private void HandleHotkey(int id)
    {
    
            if (hotkeyList["MyHotkey"] != null)
            {
                    if (id == HotkeyList["MyHotkey"].ghk.id)
                    {
                            //Do something
                    }
            }
    }

## If the user has changed hotkey settings (key binds) at runtime, re-register the hotkeys using:
    HotkeyTools.UpdateHotkeys(mainForm.HotkeyList, mainForm.HotkeyNames, mainForm);

# User rebindable hotkeys in Options form

In your Options form, add a DataGridView called HotkeyGrid. Add columns for Hotkey name, Key, and four checkboxes for the modifiers.


Get the MainForm when creating the Options form
When the form opens, fill the grid with hotkeys defined in Settings:

        //add a using statement to reference Settings.Default: using MyApp.Properties;
        
        public MainForm mainForm;
        public Options(MainForm formParent)
        {
            InitializeComponent();
            mainForm = formParent;
            // fill various other settings
            fillHotkeyGrid();
        }

        private void fillHotkeyGrid()
        {
            HotkeyGrid.Rows.Clear();
            HotkeyGrid.Rows.Add(mainForm.HotkeyList.Count);

            int i = 0;
            foreach (KeyValuePair<string, Hotkey> kvp in mainForm.HotkeyList)
            {
                string keyName = kvp.Key;
                Hotkey hotkey = kvp.Value;
                HotkeyGrid.Rows[i].Cells[0].Value = keyName;
                HotkeyGrid.Rows[i].Cells[1].Value = hotkey.Key;
                HotkeyGrid.Rows[i].Cells[2].Value = hotkey.Ctrl;
                HotkeyGrid.Rows[i].Cells[3].Value = hotkey.Alt;
                HotkeyGrid.Rows[i].Cells[4].Value = hotkey.Shift;
                HotkeyGrid.Rows[i].Cells[5].Value = hotkey.Win;
                i++;
            }
        }

When saving the Options
        
        private void saveSettings()
        {
            // saving various settings

            // save hotkey settings based on values in the HotkeyGrid
            int i = 0;
            foreach (KeyValuePair<string, Hotkey> kvp in mainForm.HotkeyList)
            {
                string keyName = kvp.Key;
                if (HotkeyGrid.Rows[i].Cells[1].Value == null)
                {
                    HotkeyGrid.Rows[i].Cells[1].Value = "";
                }

                SaveSetting("hk" + keyName + "Key", HotkeyGrid.Rows[i].Cells[1].Value.ToString()+"");
                SaveSetting("hk" + keyName + "Ctrl", Convert.ToBoolean(HotkeyGrid.Rows[i].Cells[2].Value));
                SaveSetting("hk" + keyName + "Alt", Convert.ToBoolean(HotkeyGrid.Rows[i].Cells[3].Value));
                SaveSetting("hk" + keyName + "Shift", Convert.ToBoolean(HotkeyGrid.Rows[i].Cells[4].Value));
                SaveSetting("hk" + keyName + "Win", Convert.ToBoolean(HotkeyGrid.Rows[i].Cells[5].Value));

                mainForm.HotkeyList[keyName] = GetHotkeyFromGrid(mainForm.HotkeyList[keyName], HotkeyGrid.Rows[i].Cells);

                i++;
            }

            settings.Save();
            mainForm.UpdateCapsLock(true); // updates the tray icon to a/A or normal icon

            HotkeyTools.UpdateHotkeys(mainForm.HotkeyList, mainForm.HotkeyNames, mainForm);
        }

        private static bool DoesSettingExist(string settingName)
        {
            return Settings.Default.Properties.Cast<SettingsProperty>().Any(prop => prop.Name == settingName);
        }

        private void SaveSetting(string settingName, object value)
        {
            if (DoesSettingExist(settingName))
            {
                Settings.Default[settingName] = value;
            }
            else
            {
                Debug.WriteLine("Warning. Tried to save setting that is not defined: " + settingName);
            }
        }

        private Hotkey GetHotkeyFromGrid(Hotkey hotkey, DataGridViewCellCollection settingRow)
        {
            string settingKey = string.Empty;
            DataGridViewCell cell = settingRow[1];
            if (cell != null)
            {
                if (cell.Value != null)
                    settingKey = (string)cell.Value;
                if (settingKey == null) settingKey = string.Empty;
            }

            if (settingKey.Length > 0)
                hotkey.Key = settingKey;
            else
                hotkey.Key = new string("");

            hotkey.Ctrl = Convert.ToBoolean(settingRow[2].Value);
            hotkey.Alt = Convert.ToBoolean(settingRow[3].Value);
            hotkey.Shift = Convert.ToBoolean(settingRow[4].Value);
            hotkey.Win = Convert.ToBoolean(settingRow[5].Value);

            return hotkey;
        }

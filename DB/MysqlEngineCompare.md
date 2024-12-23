# InnoDB，MyISAM 理論說明

## InnoDB

+ 優點
    + ACID 合規性：InnoDB 支持事務，確保即使在故障情況下也能保持數據完整性。這是通過提交、回滾和崩潰恢復功能實現的。
    + 行級鎖定：提供更好的並發性和寫操作性能，因為它只鎖定被更新的行，而不是整個表。
    + 外鍵支持：支持外鍵約束，強制維護表之間的參考完整性。
    + 自動崩潰恢復：使用事務日誌進行自動崩潰恢復，確保數據一致性。
    + MVCC（多版本並發控制）：允許同時進行讀寫操作而不鎖定，提高高並發環境下的性能。
+ 缺點
    + 磁盤空間：由於行級鎖定和事務日誌，通常使用更多的磁盤空間。
    + 性能：對於讀操作較重的場景，性能可能略遜於 MyISAM。
+ 建議
    + 使用 InnoDB，若無特別需要，可優先考慮使用自增鍵作為主索引

## MyISAM

+ 優點
    + 簡單性：較簡單的存儲引擎，更容易管理和配置。
    + 速度：對於讀操作較重的場景通常更快，因為它使用表級鎖定。
    + 全文搜索：內建支持全文索引和搜索，對需要文本搜索的應用程序非常有用。
    + 較少的磁盤空間使用：相比 InnoDB 通常使用更少的磁盤空間。
+ 缺點
    + 表級鎖定：在寫操作期間鎖定整個表，可能導致競爭和在寫操作較重的場景下性能變慢。
    + 無事務支持：缺乏 ACID 合規性，不支持事務，這可能導致在故障情況下數據不一致。
    + 無外鍵支持：不支持外鍵約束，因此需要在應用程序級別維護參考完整性。
    + 無自動崩潰恢復：需要手動干預進行崩潰恢復，這可能導致數據丟失。
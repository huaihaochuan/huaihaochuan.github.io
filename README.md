# huaihaochuan.github.io

Install-Package CsvHelper
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Text.RegularExpressions;
using CsvHelper;
using CsvHelper.Configuration;

public class UserRecord
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Birthday { get; set; }
    public string SubscriptionStatus { get; set; }
}

public class Program
{
    public static void Main()
    {
        var records = LoadAndCleanData("path_to_your_csv_file.csv");
        // 保存清洗后的数据或进行后续处理
    }

    private static List<UserRecord> LoadAndCleanData(string filePath)
    {
        var cleanedRecords = new List<UserRecord>();

        var config = new CsvConfiguration(CultureInfo.InvariantCulture)
        {
            HasHeaderRecord = true,
        };

        using (var reader = new StreamReader(filePath))
        using (var csv = new CsvReader(reader, config))
        {
            var records = csv.GetRecords<UserRecord>();
            foreach (var record in records)
            {
                // 1. 名字首字母大写
                record.Name = CultureInfo.CurrentCulture.TextInfo.ToTitleCase(record.Name.ToLower());

                // 2. 邮箱格式验证和修正
                record.Email = IsValidEmail(record.Email) ? record.Email : "无效的邮箱";

                // 3. 生日格式转换
                if (DateTime.TryParse(record.Birthday, out var date))
                {
                    record.Birthday = date.ToString("yyyy-MM-dd");
                }
                else
                {
                    record.Birthday = "未知";
                }

                // 4. 订阅状态检查
                record.SubscriptionStatus = (record.SubscriptionStatus == "Active" || record.SubscriptionStatus == "Inactive") ? record.SubscriptionStatus : "Inactive";

                cleanedRecords.Add(record);
            }
        }

        return cleanedRecords;
    }

    private static bool IsValidEmail(string email)
    {
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}

Для решения подходит "Шаблонный метод", так как есть общий алгоритм для получения информации о пользователе. Общий паттерн вывожу в базовый класс, а специфичные шаги отдать подклассам
Делаю абстрактный класс UserServiceTemplate, который будет содержать общий алгоритм GetUserInfo
А дальше делаю подклассы TwitterUserService и VkUserService, которые будут предоставлять конкретную реализацию для каждого шага
Убрал дублирующийся код в TwitterUserService и VkUserService

using System;
using System.Linq;
using Patterns.Ex03.ExternalLibs.Twitter;

namespace Patterns.Ex03
{
    public abstract class UserServiceTemplate
    {
        public UserInfo GetUserInfo(string pageUrl)
        {
            string userId = GetUserId(pageUrl);
            string userName = GetUserName(userId);
            var friends = GetFriends(userId);
            var convertedFriends = ConvertFriends(friends);

            return new UserInfo
            {
                UserId = userId,
                Name = userName,
                Friends = convertedFriends
            };
        }

        protected abstract string GetUserId(string pageUrl);
        protected abstract string GetUserName(string userId);
        protected abstract object[] GetFriends(string userId);
        protected abstract UserInfo[] ConvertFriends(object[] friends);
    }

    public class TwitterUserService : UserServiceTemplate
    {
        private readonly TwitterClient _client = new TwitterClient();

        protected override string GetUserId(string pageUrl)
        {
            var regex = new Regex("twitter.com/(.*)");
            var userName = regex.Match(pageUrl).Groups[0].Value;
            return _client.GetUserIdByName(userName).ToString();
        }

        protected override string GetUserName(string userId)
        {
            return _client.GetUserNameById(long.Parse(userId));
        }

        protected override object[] GetFriends(string userId)
        {
            return _client.GetSubscribers(long.Parse(userId));
        }

        protected override UserInfo[] ConvertFriends(object[] friends)
        {
            return friends.Cast<TwitterUser>()
                .Select(u => new UserInfo
                {
                    UserId = u.UserId.ToString(),
                    Name = _client.GetUserNameById(u.UserId)
                })
                .ToArray();
        }
    }

    public class VkUserService : UserServiceTemplate
    {
        protected override string GetUserId(string pageUrl)
        {
            return Parse(pageUrl);
        }

        protected override string GetUserName(string userId)
        {
            return GetName(userId);
        }

        protected override object[] GetFriends(string userId)
        {
            return GetFriendsById(userId);
        }

        protected override UserInfo[] ConvertFriends(object[] friends)
        {
            return ConvertToUserInfo(friends.Cast<VkUser>().ToArray());
        }

        private string Parse(string pageUrl) => "USER_ID";
        private string GetName(string userId) => "NAME";
        private VkUser[] GetFriendsById(string userId) => new VkUser[0];
        private UserInfo[] ConvertToUserInfo(VkUser[] friends) => new UserInfo[0];
    }
}
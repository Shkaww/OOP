Тут отлично подходит подходит паттерн Стратегия, так как  нужно обеспечить возможность добавления новых социальных сетей без изменения кода GetSubscribers. Паттерн позволяет инкапсулировать алгоритмы для разных социальных сетей в отдельные классы и динамически выбирать нужный алгоритм во время выполнения.
1.Создал интерфейс ISocialNetworkStrategy и он  будет определять метод для получения подписчиков
2.Реализовал стратегии для Twitter и Instagram чтобы адаптировать вызовы к соответствующим клиентам
3.Модифицировал класс SubscriberViewer и теперь он использует стратегии вместо прямого взаимодействия с клиентами

using System;
using Patterns.Ex02.ExternalLibs.Twitter;
using Patterns.Ex02.ExternalLibs.Instagram;

namespace Patterns.Ex02
{
    public interface ISocialNetworkStrategy
    {
        SocialNetworkUser[] GetSubscribers(string userName);
    }

    public class TwitterStrategy : ISocialNetworkStrategy
    {
        public SocialNetworkUser[] GetSubscribers(string userName)
        {
            var twitterClient = new TwitterClient();
            long userId = twitterClient.GetUserIdByName(userName);
            var subscribers = twitterClient.GetSubscribers(userId);

            var result = new SocialNetworkUser[subscribers.Length];
            for (int i = 0; i < subscribers.Length; i++)
            {
                result[i] = new SocialNetworkUser
                {
                    UserName = twitterClient.GetUserNameById(subscribers[i].UserId)
                };
            }
            return result;
        }
    }

    public class InstagramStrategy : ISocialNetworkStrategy
    {
        public SocialNetworkUser[] GetSubscribers(string userName)
        {
            var instagramClient = new InstagramClient();
            var subscribers = instagramClient.GetSubscribers(userName);

            var result = new SocialNetworkUser[subscribers.Length];
            for (int i = 0; i < subscribers.Length; i++)
            {
                result[i] = new SocialNetworkUser
                {
                    UserName = subscribers[i].UserName
                };
            }
            return result;
        }
    }

    public class SubscriberViewer
    {
        public SocialNetworkUser[] GetSubscribers(string userName, SocialNetwork networkType)
        {
            ISocialNetworkStrategy strategy;
            switch (networkType)
            {
                case SocialNetwork.Twitter:
                    strategy = new TwitterStrategy();
                    break;
                case SocialNetwork.Instagram:
                    strategy = new InstagramStrategy();
                    break;
                default:
                    throw new ArgumentException("Unsupported social network type");
            }

            return strategy.GetSubscribers(userName);
        }
    }
}